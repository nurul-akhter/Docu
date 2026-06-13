# Complete Caching Architecture for Text-to-SQL (Production Grade)

---

## Core Philosophy

| Concern | Handled by | Cost |
|---|---|---|
| Ambiguous temporal bypass | Local regex | Free |
| Temporal keywords (`today`, `last 3 years`) | Local regex + temporal dictionary | Free |
| Named entities (office, person, org) | spaCy (local pre-pass) | Free |
| Domain values (status, category, code) | Main LLM during SQL generation | Zero extra call |
| SQL generation + param extraction | Single LLM call on cache miss | Paid once per pattern |
| All subsequent value variations | Cache hit + param injection | Free |

**One LLM call per query pattern. Never two.**

---

## Overview: Pipeline

```
User Question
     │
     ▼
┌──────────────────────────┐
│  Pre-filter               │  ← Ambiguous temporal → bypass (always LLM, never cached)
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│  Temporal Extraction      │  ← Local regex + temporal dict (free, no LLM)
│  (regex + dict)           │    "last 3 years" → {time_period}, sql_expr resolved
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│  spaCy NER Pre-pass       │  ← Fast local entity detection (free)
│  (optional fast path)     │    catches GPE, PERSON, MONEY, CARDINAL…
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│  Cache Lookup             │  ← Layer 1 (Redis) → Layer 2 (pgvector)
│  by template / question   │    lookup key = LLM's query_template (not local template)
└──────────┬───────────────┘
           │ miss
           ▼
┌──────────────────────────┐
│  Single LLM Call          │  ← Returns sql_template + params + query_template
│  (SQL + param extraction) │    Retry up to 3x on JSON parse failure
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│  Validate + Execute       │  ← Inject params → run SQL → check success
└──────────┬───────────────┘
           │ success only
           ▼
┌──────────────────────────┐
│  Store in Cache           │  ← Key = LLM's query_template (authoritative)
│  (Layer 1 + Layer 2)      │    Write-through on semantic hit
└──────────────────────────┘
```

---

## 1. Pre-Filter: Only Truly Ambiguous Terms Bypass

```python
import re

AMBIGUOUS_TEMPORAL = {"recent", "recently", "latest", "just", "now"}

SENSITIVE_PATTERNS = [
    r"\bmy\b", r"\bmine\b",
    r"\bpassword\b", r"\bsecret\b",
]

def should_bypass_cache(question: str) -> tuple[bool, str]:
    q = question.lower()
    for word in AMBIGUOUS_TEMPORAL:
        if re.search(rf"\b{word}\b", q):
            return True, "ambiguous_temporal"
    for pattern in SENSITIVE_PATTERNS:
        if re.search(pattern, q):
            return True, "sensitive"
    return False, ""
```

---

## 2. Temporal Normalization: Extract `{n}` (Local, Free)

`"last 3 years"`, `"last 5 years"`, `"last 100 years"` → all share one dictionary entry.

```python
NUMERIC_TEMPORAL_PATTERN = re.compile(
    r'^(last|past|previous|next|coming|most\s+recent)\s+(\d+)\s+'
    r'(day|week|month|quarter|year|hour|minute)s?$',
    re.IGNORECASE
)

def normalize_temporal_phrase(phrase: str) -> tuple[str, dict]:
    """
    "last 3 years"        → ("last {n} years",       {"n": "3", "unit": "year"})
    "past 2 months"       → ("past {n} months",       {"n": "2", "unit": "month"})
    "most recent 4 weeks" → ("most recent {n} weeks", {"n": "4", "unit": "week"})
    "last 2 quarters"     → ("last {n} quarters",     {"n": "2", "unit": "quarter"})
    "today"               → ("today",                 {})
    """
    match = NUMERIC_TEMPORAL_PATTERN.match(phrase.strip())
    if match:
        prefix, n, unit = match.groups()
        template = f"{prefix.strip().lower()} {{n}} {unit.lower()}s"
        return template, {"n": n, "unit": unit.lower()}
    return phrase.lower(), {}
```

---

## 3. Temporal Dictionary: Static + Dynamic (LLM-Extended)

### DB Table

```sql
CREATE TABLE temporal_sql_map (
    phrase_template   TEXT PRIMARY KEY,
    sql_expr_template TEXT NOT NULL,
    source            TEXT DEFAULT 'static',
    is_valid          BOOLEAN DEFAULT TRUE,
    use_count         INT DEFAULT 0,
    created_at        TIMESTAMPTZ DEFAULT NOW(),
    last_used_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### Static Seed Data

```sql
INSERT INTO temporal_sql_map (phrase_template, sql_expr_template) VALUES
('today',              'CURRENT_DATE'),
('yesterday',          'CURRENT_DATE - INTERVAL ''1 day'''),
('tomorrow',           'CURRENT_DATE + INTERVAL ''1 day'''),
('this week',          'DATE_TRUNC(''week'',    CURRENT_DATE)'),
('last week',          'DATE_TRUNC(''week'',    CURRENT_DATE) - INTERVAL ''1 week'''),
('this month',         'DATE_TRUNC(''month'',   CURRENT_DATE)'),
('last month',         'DATE_TRUNC(''month'',   CURRENT_DATE) - INTERVAL ''1 month'''),
('this quarter',       'DATE_TRUNC(''quarter'', CURRENT_DATE)'),
('last quarter',       'DATE_TRUNC(''quarter'', CURRENT_DATE) - INTERVAL ''3 months'''),
('this year',          'DATE_TRUNC(''year'',    CURRENT_DATE)'),
('last year',          'DATE_TRUNC(''year'',    CURRENT_DATE) - INTERVAL ''1 year'''),
('last {n} days',           'CURRENT_DATE - INTERVAL ''{n} days'''),
('last {n} weeks',          'CURRENT_DATE - INTERVAL ''{n} weeks'''),
('last {n} months',         'CURRENT_DATE - INTERVAL ''{n} months'''),
('last {n} years',          'CURRENT_DATE - INTERVAL ''{n} years'''),
('last {n} quarters',       'CURRENT_DATE - INTERVAL ''{n_months} months'''),
('past {n} days',           'CURRENT_DATE - INTERVAL ''{n} days'''),
('past {n} months',         'CURRENT_DATE - INTERVAL ''{n} months'''),
('past {n} years',          'CURRENT_DATE - INTERVAL ''{n} years'''),
('past {n} quarters',       'CURRENT_DATE - INTERVAL ''{n_months} months'''),
('next {n} days',           'CURRENT_DATE + INTERVAL ''{n} days'''),
('next {n} months',         'CURRENT_DATE + INTERVAL ''{n} months'''),
('next {n} years',          'CURRENT_DATE + INTERVAL ''{n} years'''),
('most recent {n} days',    'CURRENT_DATE - INTERVAL ''{n} days'''),
('most recent {n} weeks',   'CURRENT_DATE - INTERVAL ''{n} weeks'''),
('most recent {n} months',  'CURRENT_DATE - INTERVAL ''{n} months''');
```

### `TemporalDictionary` Class

```python
class TemporalDictionary:
    def __init__(self):
        self._cache: dict[str, str] = {}
        self._load_from_db()

    def _load_from_db(self):
        with get_db_conn() as conn:
            with conn.cursor() as cur:
                cur.execute("""
                    SELECT phrase_template, sql_expr_template
                    FROM temporal_sql_map WHERE is_valid = true
                """)
                for phrase_template, sql_expr_template in cur.fetchall():
                    self._cache[phrase_template] = sql_expr_template

    def resolve(self, phrase: str) -> str | None:
        phrase_lower = phrase.lower().strip()

        if any(word in phrase_lower for word in AMBIGUOUS_TEMPORAL):
            return None

        phrase_template, temporal_params = normalize_temporal_phrase(phrase_lower)
        sql_expr_template = self._cache.get(phrase_template)

        if not sql_expr_template:
            sql_expr_template = self._resolve_via_llm(phrase_template)
            if not sql_expr_template:
                return None
            self._store(phrase_template, sql_expr_template)

        self._increment_usage(phrase_template)
        return _inject_temporal_params(sql_expr_template, temporal_params)

    def _resolve_via_llm(self, phrase_template: str) -> str | None:
        prompt = f"""Convert this temporal phrase template to a PostgreSQL SQL expression template.
Keep {{n}} as a placeholder — do NOT substitute a number.

Phrase template: "{phrase_template}"

Rules:
- Return ONLY the SQL expression template, nothing else
- Keep {{n}} in your output
- Use CURRENT_DATE, INTERVAL, DATE_TRUNC as needed
- For quarters use {{n_months}} (will be replaced with n×3)
- If ambiguous return: AMBIGUOUS

Examples:
"last {{n}} years"       → CURRENT_DATE - INTERVAL '{{n}} years'
"coming {{n}} months"    → CURRENT_DATE + INTERVAL '{{n}} months'
"most recent {{n}} days" → CURRENT_DATE - INTERVAL '{{n}} days'
"""
        response = call_llm_raw(prompt).strip()
        return None if response == "AMBIGUOUS" or not response else response

    def _store(self, phrase_template: str, sql_expr_template: str):
        self._cache[phrase_template] = sql_expr_template
        with get_db_conn() as conn:
            with conn.cursor() as cur:
                cur.execute("""
                    INSERT INTO temporal_sql_map (phrase_template, sql_expr_template, source)
                    VALUES (%s, %s, 'llm_generated')
                    ON CONFLICT (phrase_template) DO UPDATE
                        SET sql_expr_template = EXCLUDED.sql_expr_template,
                            is_valid = true
                """, (phrase_template, sql_expr_template))
            conn.commit()

    def _increment_usage(self, phrase_template: str):
        with get_db_conn() as conn:
            with conn.cursor() as cur:
                cur.execute("""
                    UPDATE temporal_sql_map
                    SET use_count = use_count + 1, last_used_at = NOW()
                    WHERE phrase_template = %s
                """, (phrase_template,))
            conn.commit()

    def invalidate(self, phrase_template: str):
        self._cache.pop(phrase_template, None)
        with get_db_conn() as conn:
            with conn.cursor() as cur:
                cur.execute("""
                    UPDATE temporal_sql_map SET is_valid = false
                    WHERE phrase_template = %s
                """, (phrase_template,))
            conn.commit()


def _inject_temporal_params(sql_expr_template: str, temporal_params: dict) -> str:
    n    = temporal_params.get("n", "")
    unit = temporal_params.get("unit", "")
    if n:
        if unit == "quarter":
            sql_expr_template = sql_expr_template.replace("{n_months}", str(int(n) * 3))
        sql_expr_template = sql_expr_template.replace("{n}", n)
    return sql_expr_template


temporal_dict = TemporalDictionary()  # singleton, loaded once at startup
```

---

## 4. Local Extraction: Temporal + spaCy (Free Pre-pass)

spaCy is a best-effort pre-pass only. Anything it misses is handled by the main LLM.
The LLM's `query_template` is always the authoritative cache key — not the local template.

```python
import spacy
nlp = spacy.load("en_core_web_sm")

TEMPORAL_PHRASE_REGEX = re.compile(
    r'\b(?:last|past|previous|next|coming|this|most\s+recent)\s+'
    r'(?:\d+\s+)?(?:day|week|month|quarter|year|hour|minute)s?\b'
    r'|\b(?:today|yesterday|tomorrow)\b',
    re.IGNORECASE
)

def extract_local(question: str) -> tuple[str, dict, bool]:
    """
    Returns: (template, params, bypass)
    Results are preliminary — LLM output takes precedence on cache miss.
    """
    template = question
    params   = {}
    bypass   = False

    # Pass 1: temporal (regex + dict — always free)
    match = TEMPORAL_PHRASE_REGEX.search(template)
    if match:
        phrase    = match.group(0)
        sql_expr  = temporal_dict.resolve(phrase)
        if sql_expr is None:
            bypass = True
        else:
            template = template.replace(phrase, "{time_period}", 1)
            params["time_period"] = phrase

    if bypass:
        return template, params, bypass

    # Pass 2: spaCy for standard named entities (fast, local)
    doc = nlp(template)
    for ent in doc.ents:
        label = ent.label_.lower()
        if label in ("date", "time"):
            continue
        params[label] = ent.text
        template = template.replace(ent.text, f"{{{label}}}")

    return template, params, bypass
```

---

## 5. Single LLM Call: SQL + Param Extraction (with Retry)

The LLM returns `sql_template`, `params`, and `query_template` in one structured call.
LLM params take priority over spaCy params to avoid label conflicts (`{gpe}` vs `{city}`).
Retries up to 3 times on JSON parse failure before falling back gracefully.

```python
import json

def call_llm(question: str) -> dict:
    prompt = f"""
{FIXED_SCHEMA_PROMPT}

User question: "{question}"

Return a JSON response with exactly these keys:
- "sql_template": parameterized SQL using {{param_name}} placeholders for ALL variable values
- "params": dict of param_name → actual value extracted from the question
- "query_template": the user question with all variable values replaced by {{param_name}} placeholders

Rules:
- Replace ALL specific filter values with placeholders: names, codes, statuses, amounts, categories
- Keep structural words as-is: total, sales, orders, by, of
- Use descriptive snake_case placeholder names
- If no variable values exist, return empty dicts and the original question as query_template

Examples:

Question: "orders with status pending above 5000 in Dhaka"
{{
  "sql_template":   "SELECT * FROM orders WHERE status='{{status}}' AND amount>{{amount}} AND city='{{city}}'",
  "params":         {{"status": "pending", "amount": "5000", "city": "Dhaka"}},
  "query_template": "orders with status {{status}} above {{amount}} in {{city}}"
}}

Question: "sales of category Electronics through channel Online"
{{
  "sql_template":   "SELECT SUM(amount) FROM sales WHERE category='{{category}}' AND channel='{{channel}}'",
  "params":         {{"category": "Electronics", "channel": "Online"}},
  "query_template": "sales of category {{category}} through channel {{channel}}"
}}

Question: "which product sells most"
{{
  "sql_template":   "SELECT product, SUM(qty) FROM sales GROUP BY product ORDER BY SUM(qty) DESC LIMIT 1",
  "params":         {{}},
  "query_template": "which product sells most"
}}
"""
    # Retry up to 3 times on JSON parse failure
    for attempt in range(3):
        try:
            response = call_llm_raw(prompt)
            return json.loads(response)
        except json.JSONDecodeError:
            if attempt == 2:
                # Fallback: treat as direct query with no params
                return {
                    "sql_template":   call_llm_sql_only(question),
                    "params":         {},
                    "query_template": question,
                }
```

---

## 6. Param Injection

LLM params take priority over spaCy params to resolve label conflicts.

```python
def inject_params(sql_template: str, params: dict) -> str:
    """
    - time_period → resolved via temporal_dict → SQL date expression
    - all others  → raw value injected directly
    """
    for key, value in params.items():
        if key == "time_period":
            sql_value = temporal_dict.resolve(value) or value
        else:
            sql_value = value
        sql_template = sql_template.replace(f"{{{key}}}", sql_value)
    return sql_template
```

---

## 7. Cache Key Design

The LLM's `query_template` is the authoritative cache key — always.
On cache lookup before the LLM call, we use the local template as a best-effort key.
On cache miss → LLM runs → we re-key using `query_template` for storage.
Layer 2 (semantic) bridges the gap between local template and LLM template via similarity.

```python
import hashlib

PROMPT_VERSION = "v1.2"  # bump whenever the fixed schema prompt changes

def make_exact_cache_key(lookup_text: str) -> str:
    normalized = normalize(lookup_text)
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

## 8. Layer 1 — Exact Match Cache (Redis)

```python
import redis, json

redis_client = redis.Redis(host="localhost", port=6379, decode_responses=True)
EXACT_TTL_SECONDS = 60 * 60 * 24 * 30  # 30 days

def exact_cache_get(lookup_text: str) -> dict | None:
    key  = make_exact_cache_key(lookup_text)
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

## 9. Layer 2 — Semantic Cache (pgvector)

```python
from pgvector.psycopg2 import register_vector
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-MiniLM-L6-v2")
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
                    "sql":          sql,
                    "has_params":   has_params_db,
                    "similarity":   float(similarity),
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
                    SET hit_count   = sql_cache.hit_count + 1,
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

### Postgres Table

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE sql_cache (
    id                  SERIAL PRIMARY KEY,
    original_question   TEXT NOT NULL,
    normalized_question TEXT NOT NULL,
    normalized_hash     TEXT UNIQUE,
    embedding           vector(384),
    has_params          BOOLEAN NOT NULL DEFAULT FALSE,
    template            TEXT,
    sql_template        TEXT,
    sql                 TEXT,
    prompt_version      TEXT NOT NULL,
    created_at          TIMESTAMPTZ DEFAULT NOW(),
    last_hit_at         TIMESTAMPTZ DEFAULT NOW(),
    hit_count           INT DEFAULT 0,
    is_valid            BOOLEAN DEFAULT TRUE,
    execution_time_ms   FLOAT
);

CREATE INDEX ON sql_cache USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
CREATE INDEX ON sql_cache (prompt_version, is_valid, has_params);
```

---

## 10. Main Pipeline

```python
import time

def text_to_sql(question: str) -> dict:
    result = {
        "question": question,
        "sql":      None,
        "source":   None,
        "error":    None,
    }

    # Step 1: ambiguous bypass check
    bypass, reason = should_bypass_cache(question)

    # Step 2: local extraction — temporal + spaCy (always free)
    local_template, local_params, temporal_bypass = extract_local(question)
    bypass = bypass or temporal_bypass

    has_params  = len(local_params) > 0
    lookup_text = local_template if has_params else question

    if not bypass:
        # Step 3: exact cache (Layer 1)
        cached = exact_cache_get(lookup_text)
        if cached:
            sql = inject_params(cached["sql_template"], local_params) \
                  if has_params else cached["sql"]
            result.update({"sql": sql, "source": "exact_cache"})
            return result

        # Step 4: semantic cache (Layer 2)
        hit = semantic_cache_get(lookup_text, has_params)
        if hit:
            sql = inject_params(hit["sql_template"], local_params) \
                  if has_params else hit["sql"]
            # Write-through using local lookup_text as key
            # Next call with same local template hits Layer 1 instantly
            exact_cache_set(lookup_text, hit)
            result.update({"sql": sql, "source": "semantic_cache"})
            return result

    # Step 5: single LLM call — SQL + param extraction (with retry)
    llm_result     = call_llm(question)
    sql_template   = llm_result["sql_template"]
    llm_params     = llm_result["params"]
    query_template = llm_result["query_template"]

    # LLM params take priority — resolves spaCy label conflicts ({gpe} vs {city})
    all_params  = {**local_params, **llm_params}
    has_params  = len(all_params) > 0
    # LLM's query_template is the authoritative cache key going forward
    lookup_text = query_template if has_params else question

    sql = inject_params(sql_template, all_params)

    # Step 6: validate by executing
    try:
        start   = time.time()
        execute_sql_safely(sql)
        exec_ms = (time.time() - start) * 1000
    except Exception as e:
        result["error"] = str(e)
        log_failed_query(question, sql, str(e))
        return result

    # Step 7: store on success, only if not bypassed
    if not bypass:
        entry = {
            "original_question":   question,
            "normalized_question": normalize(question),
            "has_params":          has_params,
            "template":            query_template if has_params else None,
            "sql_template":        sql_template   if has_params else None,
            "sql":                 sql            if not has_params else None,
            "prompt_version":      PROMPT_VERSION,
            "is_valid":            True,
            "execution_time_ms":   exec_ms,
        }
        # Store under LLM's authoritative query_template
        exact_cache_set(lookup_text, entry)
        semantic_cache_set(lookup_text, entry)

    result.update({"sql": sql, "source": "llm"})
    return result
```

---

## 11. Cache Warming (Cold Start Fix)

Seed the cache with your most frequent queries before go-live.
Eliminates the cold start LLM cost spike on first deploy.

```python
SEED_QUERIES = [
    "total sales by office",
    "top 10 customers by revenue",
    "monthly sales trend",
    "orders with status pending",
    "total revenue this year",
    # add your domain's most frequent queries
]

def warm_cache():
    print(f"Warming cache with {len(SEED_QUERIES)} seed queries...")
    for q in SEED_QUERIES:
        try:
            result = text_to_sql(q)
            status = "ok" if not result["error"] else "error"
            print(f"  [{status}] {q}")
        except Exception as e:
            print(f"  [fail] {q} — {e}")
    print("Cache warm-up complete.")

# Call during app startup, before serving traffic
# warm_cache()
```

---

## 12. User Feedback Loop (Wrong Result Correction)

Allows users to flag incorrect results so bad cache entries are removed immediately.

```python
def report_bad_result(question: str, query_template: str | None = None):
    """
    Called when a user flags a result as wrong.
    Invalidates both the specific entry and its semantic neighbours.
    """
    # Invalidate exact entry by query_template if known, else by question
    lookup = query_template if query_template else question
    invalidate_entry(lookup)
    log_bad_cache_entry(question, lookup)

def log_bad_cache_entry(question: str, lookup: str):
    with get_db_conn() as conn:
        with conn.cursor() as cur:
            cur.execute("""
                INSERT INTO cache_feedback (question, lookup_key, reported_at)
                VALUES (%s, %s, NOW())
            """, (question, lookup))
        conn.commit()
```

```sql
CREATE TABLE cache_feedback (
    id          SERIAL PRIMARY KEY,
    question    TEXT NOT NULL,
    lookup_key  TEXT NOT NULL,
    reported_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## 13. Cache Invalidation

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
    """Invalidate a specific cache entry."""
    redis_client.delete(make_exact_cache_key(lookup_text))
    with get_db_conn() as conn:
        with conn.cursor() as cur:
            cur.execute("""
                UPDATE sql_cache SET is_valid = false
                WHERE normalized_hash = %s
            """, (make_normalized_hash(lookup_text),))
        conn.commit()

def prune_old_entries(days: int = 90):
    """Remove low-traffic entries not used in N days."""
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

## 14. Monitoring

```python
from collections import defaultdict

class CacheMetrics:
    def __init__(self):
        self.counts = defaultdict(int)

    def record(self, source: str, bypass_reason: str = ""):
        self.counts[source] += 1
        if bypass_reason:
            self.counts[f"bypass_{bypass_reason}"] += 1

    def hit_rate(self) -> float:
        total = sum(v for k, v in self.counts.items()
                    if k in ("exact_cache", "semantic_cache", "llm"))
        hits  = self.counts["exact_cache"] + self.counts["semantic_cache"]
        return hits / total if total else 0.0

    def report(self) -> dict:
        return {
            "total_queries":       sum(v for k, v in self.counts.items()
                                       if k in ("exact_cache", "semantic_cache", "llm")),
            "exact_cache_hits":    self.counts["exact_cache"],
            "semantic_cache_hits": self.counts["semantic_cache"],
            "llm_calls":           self.counts["llm"],
            "bypassed_ambiguous":  self.counts["bypass_ambiguous_temporal"],
            "bypassed_sensitive":  self.counts["bypass_sensitive"],
            "cache_hit_rate":      f"{self.hit_rate():.1%}",
        }

metrics = CacheMetrics()
```

---

## 15. Full Example Flows

```
── Template key mismatch handled by Layer 2 ─────────────────────────────────

Query 1: "orders with status pending above 5000"
  local:  spaCy catches 5000 → {money}
          local template: "orders with status pending above {money}"
  Layer 1: MISS
  Layer 2: MISS
  LLM:    query_template: "orders with status {status} above {money}"  ← authoritative
           params: {status:"pending", money:"5000"}
  store:  keyed by LLM's query_template
  inject: status='pending', amount>5000  ✓

Query 2: "orders with status completed above 8000"
  local:  spaCy catches 8000 → {money}
          local template: "orders with status completed above {money}"
  Layer 1: MISS  (different from LLM's stored key — expected)
  Layer 2: HIT   (semantic similarity > 0.95 to stored template)
  write-through: local template → Layer 1
  inject: status='completed', amount>8000  ✓  NO LLM call

Query 3: "orders with status completed above 8000"  (same as query 2)
  local:  local template: "orders with status completed above {money}"
  Layer 1: HIT  (written through from query 2)  ✓  NO LLM call


── Numeric temporal + named entity ──────────────────────────────────────────

Query 1: "Dhaka office sale in last 3 years"
  local:  "last 3 years" → dict HIT → CURRENT_DATE - INTERVAL '3 years'
          "Dhaka" → spaCy GPE → {gpe}
          local template: "{gpe} office sale in {time_period}"
  Layer 1+2: MISS
  LLM:    query_template: "{city} office sale in {time_period}"
           params: {city:"Dhaka", time_period:"last 3 years"}
  store:  keyed by "{city} office sale in {time_period}"

Query 2: "Khulna office sale in last 5 years"
  local:  "last 5 years" → dict HIT, "Khulna" → GPE
          local template: "{gpe} office sale in {time_period}"
  Layer 2: HIT  (similar to "{city} office sale in {time_period}")
  inject: city='Khulna', date >= CURRENT_DATE - INTERVAL '5 years'  ✓


── Unknown temporal pattern (learned once) ──────────────────────────────────

Query 1: "sale in coming 4 weeks"
  "coming 4 weeks" → normalize → "coming {n} weeks", n=4
  temporal dict: MISS → small LLM → stores "coming {n} weeks"
  main LLM: sql_template + query_template cached
  inject: CURRENT_DATE + INTERVAL '4 weeks'  ✓

Query 2: "sale in coming 9 weeks"
  "coming 9 weeks" → "coming {n} weeks", n=9 → dict HIT
  Layer 2: HIT on query_template
  inject: CURRENT_DATE + INTERVAL '9 weeks'  ✓  NO LLM call


── Cold start (warm-up) ─────────────────────────────────────────────────────

App starts → warm_cache() runs SEED_QUERIES through full pipeline
→ cache pre-populated before first real user request
→ no cold start LLM spike


── Wrong result reported ─────────────────────────────────────────────────────

User flags "orders with status {status} above {money}" as wrong
→ report_bad_result(question, query_template)
→ invalidate_entry("orders with status {status} above {money}")
→ Redis key deleted, Postgres is_valid = false
→ next query goes to LLM, gets correct SQL, re-cached
```

---

## 16. Decision Summary

| Query | Local extract | LLM calls | Layer 1 | Layer 2 |
|---|---|---|---|---|
| "today's sale" (1st) | Temporal dict hit | 1 (SQL only) | MISS | MISS |
| "today's sale" (2nd) | Temporal dict hit | 0 | HIT | — |
| "last 5 years sale" (1st) | Temporal dict hit | 1 (SQL only) | MISS | MISS |
| "last 7 years sale" | Temporal dict hit | 0 | MISS | HIT |
| "coming 4 weeks" (1st) | Temporal dict miss | 1 (temporal) + 1 (SQL) | MISS | MISS |
| "coming 9 weeks" | Temporal dict hit | 0 | MISS | HIT |
| "Dhaka office sale" (1st) | spaCy hit | 1 (SQL only) | MISS | MISS |
| "Khulna office sale" | spaCy hit | 0 | MISS | HIT |
| "status pending orders" (1st) | spaCy partial | 1 (SQL + params) | MISS | MISS |
| "status completed orders" | spaCy partial | 0 | MISS | HIT |
| Same query repeated | Any | 0 | HIT | — |
| "recent sale" | Ambiguous bypass | 1 (always) | Never | Never |
| Bad result reported | — | 1 (re-generates) | Cleared | Cleared |
| Cold start (seeded) | — | 1 per seed | Warm | Warm |

---

## 17. Recommended Stack

| Component | Choice | Why |
|---|---|---|
| Exact cache | Redis | Sub-millisecond, TTL built-in |
| Vector store | pgvector on Postgres | Single DB, production-proven |
| Temporal dict | Postgres `temporal_sql_map` | Persists LLM-learned patterns |
| Feedback log | Postgres `cache_feedback` | Audit trail for bad results |
| Embeddings | `all-MiniLM-L6-v2` (local) | No API cost per request |
| Entity extraction | spaCy `en_core_web_sm` | Fast, local, free pre-pass |
| LLM (main SQL) | Claude Sonnet / GPT-4o | Cache miss only, structured JSON |
| LLM (temporal) | Claude Haiku / GPT-4o-mini | Cheap, once per unknown pattern |
| Similarity threshold | Start 0.95, tune cautiously | Safety over hit rate |

---

> **Golden rules:**
> 1. Cache SQL templates, never query results.
> 2. One LLM call per query **pattern**, never per query **value**.
> 3. Temporal patterns are resolved locally — LLM learns unknown patterns once, never again.
> 4. The main LLM always handles SQL + param extraction together — never two calls.
> 5. LLM's `query_template` is the authoritative cache key — local template is best-effort only.
> 6. Layer 2 (semantic) bridges key mismatches between local and LLM templates.
> 7. Warm the cache before go-live. Give users a way to report wrong results.
