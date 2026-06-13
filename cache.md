# Complete Caching Architecture for Text-to-SQL (Production Grade)

---

## Overview: Two-Track, Three-Layer Pipeline

```
User Question
     │
     ▼
┌──────────────────────┐
│  Pre-filter           │  ← Temporal/sensitive bypass → always LLM
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  Entity Extraction    │  ← spaCy: detect named values (office, amount, limit…)
│  + Templatization     │
└──────────┬───────────┘
           │
     has params?
    ┌──────┴──────┐
   YES            NO
    │              │
    ▼              ▼
TEMPLATE       DIRECT
TRACK          TRACK
    │              │
    ▼              ▼
Layer 1:       Layer 1:
Exact cache    Exact cache
(Redis)        (Redis)
    │ miss          │ miss
    ▼              ▼
Layer 2:       Layer 2:
Semantic on    Semantic on
TEMPLATE       full question
(pgvector)     (pgvector)
    │ hit           │ hit
    ▼              ▼
inject         return
params         SQL as-is
    │              │
    └──────┬───────┘
           │ miss (both tracks)
           ▼
      Layer 3: LLM call
           │
           ▼
      Validate (execute)
           │ success only
           ▼
      Store in Layer 1 + Layer 2
      (as template if has params,
       as-is if no params)
```

---

## 1. Pre-Filter: What Never Goes to Cache

```python
import re

TEMPORAL_PATTERNS = [
    r"\btoday\b", r"\byesterday\b", r"\btomorrow\b",
    r"\bthis (week|month|year|quarter)\b",
    r"\blast (week|month|year|quarter)\b",
    r"\bnext (week|month|year)\b",
    r"\brecent(ly)?\b", r"\blatest\b", r"\bnow\b",
    r"\bcurrent(ly)?\b", r"\bjust\b",
    r"\b\d+ (days?|hours?|minutes?) ago\b",
]

SENSITIVE_PATTERNS = [
    r"\bmy\b", r"\bmine\b",          # user-specific intent
    r"\bpassword\b", r"\bsecret\b",  # sensitive fields
]

def should_bypass_cache(question: str) -> tuple[bool, str]:
    q = question.lower()
    for pattern in TEMPORAL_PATTERNS:
        if re.search(pattern, q):
            return True, "temporal"
    for pattern in SENSITIVE_PATTERNS:
        if re.search(pattern, q):
            return True, "sensitive"
    return False, ""
```

---

## 2. Entity Extraction and Templatization

This is the core of the production approach. Named values (offices, amounts, limits,
product names) are extracted and replaced with typed placeholders before caching.

```python
import spacy
nlp = spacy.load("en_core_web_sm")

def extract_and_templatize(question: str) -> tuple[str, dict]:
    """
    Returns:
        template: question with entity values replaced by {label} placeholders
        params:   dict of {label: original_value}

    Example:
        "total sales of Dhaka office above 50000"
        → template: "total sales of {gpe} office above {money}"
        → params:   {"gpe": "Dhaka", "money": "50000"}
    """
    doc = nlp(question)
    params = {}
    template = question

    for ent in doc.ents:
        placeholder = f"{{{ent.label_.lower()}}}"
        params[ent.label_.lower()] = ent.text
        template = template.replace(ent.text, placeholder)

    return template, params


def inject_params(sql_template: str, params: dict) -> str:
    """Replace {label} placeholders in SQL template with actual values."""
    for key, value in params.items():
        sql_template = sql_template.replace(f"{{{key}}}", value)
    return sql_template
```

**Examples of what gets extracted:**

| Query | Template | Params |
|---|---|---|
| "total sales of Dhaka office" | "total sales of {gpe} office" | `{gpe: "Dhaka"}` |
| "total sales of Khulna office" | "total sales of {gpe} office" | `{gpe: "Khulna"}` |
| "sales above 50000 taka" | "sales above {money} taka" | `{money: "50000"}` |
| "top 5 customers by revenue" | "top {cardinal} customers by revenue" | `{cardinal: "5"}` |
| "which product sells most" | "which product sells most" | `{}` — no params |
| "compare sales across all offices" | "compare sales across all offices" | `{}` — no params |

---

## 3. Cache Key Design

Schema versioning = prompt versioning, since the schema lives in a fixed prompt.

```python
import hashlib

PROMPT_VERSION = "v1.2"  # bump whenever you update the fixed schema prompt

def make_exact_cache_key(text: str) -> str:
    normalized = normalize(text)
    raw = f"{PROMPT_VERSION}::{normalized}"
    return f"t2sql:exact:{hashlib.sha256(raw.encode()).hexdigest()}"

def make_normalized_hash(text: str) -> str:
    return hashlib.sha256(normalize(text).encode()).hexdigest()

def normalize(text: str) -> str:
    q = text.lower().strip()
    q = re.sub(r"[^\w\s]", "", q)
    q = re.sub(r"\s+", " ", q)
    return q
```

---

## 4. Cache Entry Schema

```python
from dataclasses import dataclass
from datetime import datetime

@dataclass
class CacheEntry:
    # Identity
    original_question: str
    normalized_question: str
    embedding: list[float]

    # Template track fields
    has_params: bool
    template: str | None          # "total sales of {gpe} office"
    sql_template: str | None      # SELECT ... WHERE office = '{gpe}'

    # Direct track fields
    sql: str | None               # populated only when has_params=False

    # Metadata
    prompt_version: str
    created_at: datetime
    last_hit_at: datetime
    hit_count: int
    is_valid: bool
    execution_time_ms: float
    confidence: float             # similarity score (1.0 for exact hits)
```

---

## 5. Layer 1 — Exact Match Cache (Redis)

For **template track**: key is built from the normalized **template**.
For **direct track**: key is built from the normalized **question**.

```python
import redis
import json

redis_client = redis.Redis(host="localhost", port=6379, decode_responses=True)
EXACT_TTL_SECONDS = 60 * 60 * 24 * 30  # 30 days

def exact_cache_get(lookup_text: str) -> dict | None:
    key = make_exact_cache_key(lookup_text)
    data = redis_client.get(key)
    if data:
        entry = json.loads(data)
        if entry["is_valid"] and entry["prompt_version"] == PROMPT_VERSION:
            redis_client.expire(key, EXACT_TTL_SECONDS)
            return entry
    return None

def exact_cache_set(lookup_text: str, entry: dict):
    key = make_exact_cache_key(lookup_text)
    redis_client.setex(key, EXACT_TTL_SECONDS, json.dumps(entry))
```

---

## 6. Layer 2 — Semantic Cache (pgvector)

For **template track**: embed and search by **template string**.
For **direct track**: embed and search by **full question**.

```python
from pgvector.psycopg2 import register_vector
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-MiniLM-L6-v2")  # local, no API call
SIMILARITY_THRESHOLD = 0.95

def embed(text: str) -> list[float]:
    return model.encode(normalize(text)).tolist()

def semantic_cache_get(lookup_text: str, has_params: bool) -> dict | None:
    embedding = embed(lookup_text)

    with get_db_conn() as conn:
        register_vector(conn)
        with conn.cursor() as cur:
            cur.execute("""
                SELECT sql_template, sql, has_params,
                       1 - (embedding <=> %s::vector) AS similarity
                FROM sql_cache
                WHERE is_valid = true
                  AND prompt_version = %s
                  AND has_params = %s
                  AND 1 - (embedding <=> %s::vector) >= %s
                ORDER BY similarity DESC
                LIMIT 1
            """, (embedding, PROMPT_VERSION, has_params, embedding, SIMILARITY_THRESHOLD))

            row = cur.fetchone()
            if row:
                sql_template, sql, has_params_db, similarity = row
                return {
                    "sql_template": sql_template,
                    "sql": sql,
                    "has_params": has_params_db,
                    "similarity": float(similarity),
                }
    return None

def semantic_cache_set(lookup_text: str, entry: dict):
    embedding = embed(lookup_text)

    with get_db_conn() as conn:
        register_vector(conn)
        with conn.cursor() as cur:
            cur.execute("""
                INSERT INTO sql_cache
                    (original_question, normalized_question, normalized_hash,
                     embedding, has_params, template, sql_template, sql,
                     prompt_version, execution_time_ms)
                VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
                ON CONFLICT (normalized_hash) DO UPDATE
                    SET hit_count  = sql_cache.hit_count + 1,
                        last_hit_at = NOW()
            """, (
                entry["original_question"],
                entry["normalized_question"],
                make_normalized_hash(lookup_text),
                embedding,
                entry["has_params"],
                entry.get("template"),
                entry.get("sql_template"),
                entry.get("sql"),
                PROMPT_VERSION,
                entry.get("execution_time_ms", 0),
            ))
        conn.commit()
```

**Postgres table:**

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE sql_cache (
    id                  SERIAL PRIMARY KEY,
    original_question   TEXT NOT NULL,
    normalized_question TEXT NOT NULL,
    normalized_hash     TEXT UNIQUE,
    embedding           vector(384),
    has_params          BOOLEAN NOT NULL DEFAULT FALSE,
    template            TEXT,               -- "total sales of {gpe} office"
    sql_template        TEXT,               -- SELECT ... WHERE office = '{gpe}'
    sql                 TEXT,               -- populated when has_params = false
    prompt_version      TEXT NOT NULL,
    created_at          TIMESTAMPTZ DEFAULT NOW(),
    last_hit_at         TIMESTAMPTZ DEFAULT NOW(),
    hit_count           INT DEFAULT 0,
    is_valid            BOOLEAN DEFAULT TRUE,
    execution_time_ms   FLOAT
);

CREATE INDEX ON sql_cache USING ivfflat (embedding vector_cosine_ops)
    WITH (lists = 100);

CREATE INDEX ON sql_cache (prompt_version, is_valid, has_params);
```

---

## 7. Main Pipeline

```python
import time

def text_to_sql(question: str) -> dict:
    result = {
        "question": question,
        "sql": None,
        "source": None,      # "exact_cache" | "semantic_cache" | "llm"
        "similarity": None,
        "error": None,
    }

    # Step 1: bypass check
    bypass, reason = should_bypass_cache(question)

    # Step 2: extract entities and templatize
    template, params = extract_and_templatize(question)
    has_params = len(params) > 0

    # Template track uses template as lookup key; direct track uses full question
    lookup_text = template if has_params else question

    if not bypass:
        # Step 3: exact cache (Layer 1)
        cached = exact_cache_get(lookup_text)
        if cached:
            sql = inject_params(cached["sql_template"], params) if has_params else cached["sql"]
            result.update({"sql": sql, "source": "exact_cache", "similarity": 1.0})
            return result

        # Step 4: semantic cache (Layer 2)
        hit = semantic_cache_get(lookup_text, has_params)
        if hit:
            sql = inject_params(hit["sql_template"], params) if has_params else hit["sql"]
            # Write-through to exact cache for instant future hits
            exact_cache_set(lookup_text, hit)
            result.update({"sql": sql, "source": "semantic_cache", "similarity": hit["similarity"]})
            return result

    # Step 5: LLM call (Layer 3)
    if has_params:
        # Ask LLM to generate parameterized SQL using placeholders
        sql_template = call_llm_for_template(question, template, params)
        sql = inject_params(sql_template, params)
    else:
        sql = call_llm(question)
        sql_template = None

    # Step 6: validate by executing
    try:
        start = time.time()
        execute_sql_safely(sql)
        exec_ms = (time.time() - start) * 1000
    except Exception as e:
        result["error"] = str(e)
        log_failed_query(question, sql, str(e))
        return result

    # Step 7: store in cache only on success, only if not bypassed
    if not bypass:
        entry = {
            "original_question": question,
            "normalized_question": normalize(question),
            "has_params": has_params,
            "template": template if has_params else None,
            "sql_template": sql_template,
            "sql": sql if not has_params else None,
            "prompt_version": PROMPT_VERSION,
            "is_valid": True,
            "execution_time_ms": exec_ms,
        }
        exact_cache_set(lookup_text, entry)
        semantic_cache_set(lookup_text, entry)

    result.update({"sql": sql, "source": "llm"})
    return result
```

---

## 8. LLM Prompt for Template Generation

When `has_params=True`, instruct the LLM to use the placeholder in the SQL:

```python
def call_llm_for_template(question: str, template: str, params: dict) -> str:
    placeholder_note = "\n".join(
        f"- '{v}' has been abstracted as {{{k}}} — use {{{k}}} as a placeholder in the SQL"
        for k, v in params.items()
    )
    prompt = f"""
{FIXED_SCHEMA_PROMPT}

User question: {question}
Templatized question: {template}

The following values are parameters — use their placeholder names in the SQL instead of hardcoded values:
{placeholder_note}

Generate a parameterized SQL query using the placeholders above.
"""
    return call_llm_raw(prompt)
```

---

## 9. Cache Invalidation Strategy

```python
def invalidate_by_prompt_version():
    """Call whenever the fixed schema prompt is updated."""
    with get_db_conn() as conn:
        with conn.cursor() as cur:
            cur.execute("""
                UPDATE sql_cache SET is_valid = false
                WHERE prompt_version != %s
            """, (PROMPT_VERSION,))
        conn.commit()

def invalidate_entry(lookup_text: str):
    """Manually invalidate a known bad cached entry."""
    redis_client.delete(make_exact_cache_key(lookup_text))

    with get_db_conn() as conn:
        with conn.cursor() as cur:
            cur.execute("""
                UPDATE sql_cache SET is_valid = false
                WHERE normalized_hash = %s
            """, (make_normalized_hash(lookup_text),))
        conn.commit()

def prune_old_entries(days: int = 90):
    """Remove low-traffic entries not hit in N days."""
    with get_db_conn() as conn:
        with conn.cursor() as cur:
            cur.execute("""
                DELETE FROM sql_cache
                WHERE last_hit_at < NOW() - INTERVAL '%s days'
                  AND hit_count < 5
            """, (days,))
        conn.commit()
```

---

## 10. Monitoring

```python
from collections import defaultdict

class CacheMetrics:
    def __init__(self):
        self.counts = defaultdict(int)

    def record(self, source: str, track: str = "", bypass_reason: str = ""):
        self.counts[source] += 1
        if track:
            self.counts[f"{source}_{track}"] += 1
        if bypass_reason:
            self.counts[f"bypass_{bypass_reason}"] += 1

    def hit_rate(self) -> float:
        total = sum(v for k, v in self.counts.items()
                    if k in ("exact_cache", "semantic_cache", "llm"))
        hits = self.counts["exact_cache"] + self.counts["semantic_cache"]
        return hits / total if total else 0.0

    def report(self) -> dict:
        return {
            "total_queries":          sum(v for k, v in self.counts.items()
                                          if k in ("exact_cache", "semantic_cache", "llm")),
            "exact_cache_hits":       self.counts["exact_cache"],
            "exact_template_hits":    self.counts["exact_cache_template"],
            "exact_direct_hits":      self.counts["exact_cache_direct"],
            "semantic_cache_hits":    self.counts["semantic_cache"],
            "semantic_template_hits": self.counts["semantic_cache_template"],
            "semantic_direct_hits":   self.counts["semantic_cache_direct"],
            "llm_calls":              self.counts["llm"],
            "bypassed_temporal":      self.counts["bypass_temporal"],
            "bypassed_sensitive":     self.counts["bypass_sensitive"],
            "cache_hit_rate":         f"{self.hit_rate():.1%}",
        }

metrics = CacheMetrics()
```

---

## 11. Decision Summary

| Scenario | Track | Cache behavior |
|---|---|---|
| "total sales of Dhaka office" (1st) | Template | LLM → store template + SQL template |
| "total sales of Khulna office" (1st) | Template | Semantic HIT on template → inject Khulna |
| "total sales of Khulna office" (2nd) | Template | Exact HIT on template → inject Khulna |
| "which product sells most" (1st) | Direct | LLM → store SQL as-is |
| "which product sells most" (2nd) | Direct | Exact HIT → return SQL |
| "what product has highest sales" | Direct | Semantic HIT (similar intent) |
| "today's sales of Dhaka office" | — | Bypassed (temporal) → always LLM |
| Failed SQL from LLM | — | Never stored |
| Prompt/schema updated | — | Bump `PROMPT_VERSION` → old entries rejected |
| Not used in 90 days + low hits | — | Pruned by maintenance job |

---

## 12. Recommended Stack

| Component | Choice | Why |
|---|---|---|
| Exact cache | Redis | Sub-millisecond, TTL built-in |
| Vector store | pgvector on Postgres | Single DB, production-proven |
| Embeddings | `all-MiniLM-L6-v2` (local) | No extra API cost per request |
| Entity extraction | spaCy `en_core_web_sm` | Fast, local, no API call |
| LLM | Claude Sonnet / GPT-4o | Cache miss only |
| Similarity threshold | Start at 0.95, tune down cautiously | Safety over hit rate |

---

> **Golden rule:** Cache SQL templates, never query results.
> One LLM call per query **pattern** — not per query **value**.
