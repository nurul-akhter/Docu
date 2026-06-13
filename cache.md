# Complete Caching Architecture for Text-to-SQL (Production Grade)

---

## Overview: Two-Track, Three-Layer Pipeline

```
User Question
     │
     ▼
┌──────────────────────────┐
│  Pre-filter               │  ← Ambiguous temporal bypass → always LLM
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│  Temporal Normalization   │  ← Extract {n} from "last 3 years" → "last {n} years"
│  + Temporal Dict Resolve  │  ← Static hit / DB hit / LLM resolve (once per pattern)
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│  Entity Extraction        │  ← spaCy: office, product, amount, limit…
│  + Templatization         │
└──────────┬───────────────┘
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

## 1. Pre-Filter: Only Truly Ambiguous Terms Bypass

Deterministic temporal terms (today, last 3 years, past 2 months) are **NOT** bypassed —
they are resolved via the temporal dictionary. Only terms with no deterministic SQL
equivalent are bypassed.

```python
import re

# These have no deterministic SQL equivalent — bypass cache entirely
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

## 2. Temporal Normalization: Extract `{n}`

Converts a specific temporal phrase into a reusable template by extracting its numeric value.
This means `"last 3 years"`, `"last 5 years"`, `"last 100 years"` all share **one** dictionary entry.

```python
NUMERIC_TEMPORAL_PATTERN = re.compile(
    r'^(last|past|previous|next|coming|most\s+recent)\s+(\d+)\s+'
    r'(day|week|month|quarter|year|hour|minute)s?$',
    re.IGNORECASE
)

def normalize_temporal_phrase(phrase: str) -> tuple[str, dict]:
    """
    "last 3 years"        → ("last {n} years",         {"n": "3", "unit": "year"})
    "past 2 months"       → ("past {n} months",         {"n": "2", "unit": "month"})
    "most recent 4 weeks" → ("most recent {n} weeks",   {"n": "4", "unit": "week"})
    "last 2 quarters"     → ("last {n} quarters",       {"n": "2", "unit": "quarter"})
    "last week"           → ("last week",               {})   ← no number
    "today"               → ("today",                   {})
    """
    match = NUMERIC_TEMPORAL_PATTERN.match(phrase.strip())
    if match:
        prefix, n, unit = match.groups()
        prefix = prefix.strip().lower()
        template = f"{prefix} {{n}} {unit.lower()}s"
        return template, {"n": n, "unit": unit.lower()}
    return phrase.lower(), {}
```

---

## 3. Temporal Dictionary: Static + Dynamic (LLM-Extended)

### DB Table

```sql
CREATE TABLE temporal_sql_map (
    phrase_template  TEXT PRIMARY KEY,      -- "last {n} years"
    sql_expr_template TEXT NOT NULL,        -- "CURRENT_DATE - INTERVAL '{n} years'"
    source           TEXT DEFAULT 'static', -- 'static' | 'llm_generated'
    is_valid         BOOLEAN DEFAULT TRUE,
    use_count        INT DEFAULT 0,
    created_at       TIMESTAMPTZ DEFAULT NOW(),
    last_used_at     TIMESTAMPTZ DEFAULT NOW()
);
```

### Static Seed Data

```sql
INSERT INTO temporal_sql_map (phrase_template, sql_expr_template) VALUES
-- Fixed phrases
('today',           'CURRENT_DATE'),
('yesterday',       'CURRENT_DATE - INTERVAL ''1 day'''),
('tomorrow',        'CURRENT_DATE + INTERVAL ''1 day'''),
('this week',       'DATE_TRUNC(''week'', CURRENT_DATE)'),
('last week',       'DATE_TRUNC(''week'', CURRENT_DATE) - INTERVAL ''1 week'''),
('this month',      'DATE_TRUNC(''month'', CURRENT_DATE)'),
('last month',      'DATE_TRUNC(''month'', CURRENT_DATE) - INTERVAL ''1 month'''),
('this quarter',    'DATE_TRUNC(''quarter'', CURRENT_DATE)'),
('last quarter',    'DATE_TRUNC(''quarter'', CURRENT_DATE) - INTERVAL ''3 months'''),
('this year',       'DATE_TRUNC(''year'', CURRENT_DATE)'),
('last year',       'DATE_TRUNC(''year'', CURRENT_DATE) - INTERVAL ''1 year'''),
-- Dynamic templates (cover any value of n)
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
        """
        Full resolution pipeline:
          1. Normalize phrase → extract {n}
          2. Lookup phrase template in dict
          3. If miss → LLM resolves the TEMPLATE (learned once, covers all n)
          4. Inject {n} into SQL expression
        """
        phrase_lower = phrase.lower().strip()

        if any(word in phrase_lower for word in AMBIGUOUS_TEMPORAL):
            return None

        # Step 1: normalize → extract {n}
        phrase_template, temporal_params = normalize_temporal_phrase(phrase_lower)

        # Step 2: lookup by template
        sql_expr_template = self._cache.get(phrase_template)

        # Step 3: LLM resolution for unknown templates
        if not sql_expr_template:
            sql_expr_template = self._resolve_via_llm(phrase_template)
            if not sql_expr_template:
                return None
            self._store(phrase_template, sql_expr_template)

        self._increment_usage(phrase_template)

        # Step 4: inject {n} into SQL expression template
        return inject_temporal_params(sql_expr_template, temporal_params)

    def _resolve_via_llm(self, phrase_template: str) -> str | None:
        prompt = f"""You are a SQL expression generator for temporal phrase templates.

Convert this temporal phrase template to a PostgreSQL SQL expression template.
Keep {{n}} as a placeholder in your output — do NOT substitute a number.

Phrase template: "{phrase_template}"

Rules:
- Return ONLY the SQL expression template, nothing else
- Keep {{n}} as a placeholder in your output
- Use CURRENT_DATE, INTERVAL, DATE_TRUNC as needed
- For quarters use {{n_months}} (will be replaced with n×3)
- If the phrase is too ambiguous, return: AMBIGUOUS

Examples:
"last {{n}} years"          → CURRENT_DATE - INTERVAL '{{n}} years'
"past {{n}} months"         → CURRENT_DATE - INTERVAL '{{n}} months'
"next {{n}} weeks"          → CURRENT_DATE + INTERVAL '{{n}} weeks'
"most recent {{n}} days"    → CURRENT_DATE - INTERVAL '{{n}} days'
"coming {{n}} months"       → CURRENT_DATE + INTERVAL '{{n}} months'
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


temporal_dict = TemporalDictionary()  # singleton, loaded once at startup
```

---

## 4. Temporal Param Injection

```python
def inject_temporal_params(sql_expr_template: str, temporal_params: dict) -> str:
    """
    Injects {n} into the SQL expression template.
    Handles special unit conversions (quarters → months).
    """
    n = temporal_params.get("n", "")
    unit = temporal_params.get("unit", "")

    if n:
        if unit == "quarter":
            n_months = str(int(n) * 3)
            sql_expr_template = sql_expr_template.replace("{n_months}", n_months)
        sql_expr_template = sql_expr_template.replace("{n}", n)

    return sql_expr_template
```

---

## 5. Entity Extraction + Templatization

```python
import spacy
nlp = spacy.load("en_core_web_sm")

TEMPORAL_PHRASE_REGEX = re.compile(
    r'\b(?:last|past|previous|next|coming|this|most\s+recent)\s+'
    r'(?:\d+\s+)?(?:day|week|month|quarter|year|hour|minute)s?\b'
    r'|\b(?:today|yesterday|tomorrow)\b',
    re.IGNORECASE
)

def extract_and_templatize(question: str) -> tuple[str, dict, bool]:
    """
    Returns: (template, params, bypass)
    bypass=True when temporal phrase is ambiguous — skip cache entirely.

    Params dict contains:
      - time_period: original temporal phrase (resolved separately via temporal_dict)
      - entity labels: named entity values (office, product, etc.)
    """
    template = question
    params = {}
    bypass = False

    # Pass 1: detect and resolve temporal phrase
    match = TEMPORAL_PHRASE_REGEX.search(template)
    if match:
        phrase = match.group(0)
        sql_expr = temporal_dict.resolve(phrase)

        if sql_expr is None:
            bypass = True
        else:
            template = template.replace(phrase, "{time_period}", 1)
            params["time_period"] = phrase  # original phrase stored; SQL injected later

    # Pass 2: spaCy for named entities
    doc = nlp(template)
    for ent in doc.ents:
        label = ent.label_.lower()
        if label in ("date", "time"):
            continue   # already handled in Pass 1
        placeholder = f"{{{label}}}"
        params[label] = ent.text
        template = template.replace(ent.text, placeholder)

    return template, params, bypass
```

---

## 6. Entity + Temporal Param Injection

```python
def inject_params(sql_template: str, params: dict) -> str:
    """
    - time_period → resolved via temporal_dict → SQL expression
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
from datetime import datetime

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

## 9. Layer 2 — Semantic Cache (pgvector)

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
    template            TEXT,               -- "what is {time_period} sale of {gpe}"
    sql_template        TEXT,               -- SELECT ... WHERE date >= {time_period} AND office='{gpe}'
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

## 10. Main Pipeline

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

    # Step 1: ambiguous bypass check
    bypass, reason = should_bypass_cache(question)

    # Step 2: extract entities + templatize (resolves temporal dict internally)
    template, params, temporal_bypass = extract_and_templatize(question)
    bypass = bypass or temporal_bypass

    has_params = len(params) > 0
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
            exact_cache_set(lookup_text, hit)  # write-through for next identical query
            result.update({"sql": sql, "source": "semantic_cache", "similarity": hit["similarity"]})
            return result

    # Step 5: LLM call (Layer 3)
    if has_params:
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

    # Step 7: store on success, only if not bypassed
    if not bypass:
        entry = {
            "original_question":   question,
            "normalized_question": normalize(question),
            "has_params":          has_params,
            "template":            template if has_params else None,
            "sql_template":        sql_template,
            "sql":                 sql if not has_params else None,
            "prompt_version":      PROMPT_VERSION,
            "is_valid":            True,
            "execution_time_ms":   exec_ms,
        }
        exact_cache_set(lookup_text, entry)
        semantic_cache_set(lookup_text, entry)

    result.update({"sql": sql, "source": "llm"})
    return result
```

---

## 11. LLM Prompt for Template Generation

```python
def call_llm_for_template(question: str, template: str, params: dict) -> str:
    placeholder_note = "\n".join(
        f"- '{v}' is abstracted as {{{k}}} — use {{{k}}} as placeholder in SQL"
        for k, v in params.items()
    )
    prompt = f"""
{FIXED_SCHEMA_PROMPT}

User question: {question}
Templatized question: {template}

The following values are parameters — use their placeholder names in the SQL:
{placeholder_note}

Generate a parameterized SQL query using the placeholders above.
For time_period, the placeholder will be replaced with the correct SQL date expression at runtime.
"""
    return call_llm_raw(prompt)
```

---

## 12. Cache Invalidation

```python
def invalidate_by_prompt_version():
    with get_db_conn() as conn:
        with conn.cursor() as cur:
            cur.execute("""
                UPDATE sql_cache SET is_valid = false
                WHERE prompt_version != %s
            """, (PROMPT_VERSION,))
        conn.commit()

def invalidate_entry(lookup_text: str):
    redis_client.delete(make_exact_cache_key(lookup_text))
    with get_db_conn() as conn:
        with conn.cursor() as cur:
            cur.execute("""
                UPDATE sql_cache SET is_valid = false
                WHERE normalized_hash = %s
            """, (make_normalized_hash(lookup_text),))
        conn.commit()

def prune_old_entries(days: int = 90):
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

## 13. Monitoring

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
            "bypassed_ambiguous":     self.counts["bypass_ambiguous_temporal"],
            "bypassed_sensitive":     self.counts["bypass_sensitive"],
            "cache_hit_rate":         f"{self.hit_rate():.1%}",
        }

metrics = CacheMetrics()
```

---

## 14. Full Example Flows

```
Query: "sale in last 3 years"
  normalize_temporal_phrase("last 3 years") → template="last {n} years", n="3"
  temporal_dict lookup → STATIC HIT
  sql_expr_template = "CURRENT_DATE - INTERVAL '{n} years'"
  inject n=3 → "CURRENT_DATE - INTERVAL '3 years'"
  extract_and_templatize → template="sale in {time_period}", params={time_period:"last 3 years"}
  sql_cache lookup → MISS
  LLM → sql_template: "SELECT SUM(amt) FROM sales WHERE date >= {time_period}"
  inject → "WHERE date >= CURRENT_DATE - INTERVAL '3 years'"  ✓  stored

Query: "sale in last 5 years"
  normalize → template="last {n} years", n="5"  ← same pattern
  temporal_dict → STATIC HIT → inject n=5
  extract_and_templatize → template="sale in {time_period}"  ← same template!
  sql_cache → SEMANTIC HIT
  inject → "WHERE date >= CURRENT_DATE - INTERVAL '5 years'"  ✓  NO LLM call

Query: "sale in last 2 quarters"
  normalize → template="last {n} quarters", n="2", unit="quarter"
  temporal_dict → STATIC HIT → inject_temporal_params: n_months=2×3=6
  sql_expr = "CURRENT_DATE - INTERVAL '6 months'"
  sql_cache → SEMANTIC HIT (same template structure)
  inject → correct SQL  ✓  NO LLM call

Query: "sale in coming 4 weeks"
  normalize → template="coming {n} weeks", n="4"
  temporal_dict → MISS → LLM resolves template → "CURRENT_DATE + INTERVAL '{n} weeks'"
  stored in DB as "coming {n} weeks"  ← learned once
  inject n=4 → "CURRENT_DATE + INTERVAL '4 weeks'"  ✓

Query: "sale in coming 9 weeks"
  normalize → template="coming {n} weeks", n="9"
  temporal_dict → DB HIT  ← no LLM
  inject n=9 → "CURRENT_DATE + INTERVAL '9 weeks'"  ✓

Query: "what is recent sale"
  "recent" in AMBIGUOUS_TEMPORAL → bypass → full LLM → NOT cached
```

---

## 15. Decision Summary

| Query | Temporal type | n dynamic | Cache behavior |
|---|---|---|---|
| "today's sale" | Static phrase | — | Template HIT → inject CURRENT_DATE |
| "last week's sale" | Static phrase | — | Template HIT → inject DATE_TRUNC |
| "last 3 years sale" | Static template | yes | Template HIT → inject n=3 |
| "last 5 years sale" | Static template | yes | Template HIT → inject n=5 |
| "last 2 quarters sale" | Static template | yes (×3) | Template HIT → inject n_months=6 |
| "coming 4 weeks sale" | LLM template (1st) | yes | LLM resolves template, stored, inject n=4 |
| "coming 9 weeks sale" | LLM template (2nd) | yes | DB HIT → inject n=9, no LLM |
| "Dhaka office sale" | Named entity | — | Template HIT → inject "Dhaka" |
| "recent sale" | Ambiguous | — | Bypass → always LLM, never cached |

---

## 16. Recommended Stack

| Component | Choice | Why |
|---|---|---|
| Exact cache | Redis | Sub-millisecond, TTL built-in |
| Vector store | pgvector on Postgres | Single DB, production-proven |
| Temporal dict store | Postgres (`temporal_sql_map`) | Persists LLM-learned patterns |
| Embeddings | `all-MiniLM-L6-v2` (local) | No extra API cost per request |
| Entity extraction | spaCy `en_core_web_sm` | Fast, local, no API call |
| LLM (main) | Claude Sonnet / GPT-4o | Cache miss only |
| LLM (temporal) | Claude Haiku / GPT-4o-mini | Cheap, short prompt, called once per new pattern |
| Similarity threshold | Start at 0.95, tune cautiously | Safety over hit rate |

---

> **Golden rules:**
> 1. Cache SQL templates, never query results.
> 2. One LLM call per query **pattern**, not per query **value**.
> 3. One temporal LLM call per **phrase pattern**, not per **numeric value**.
