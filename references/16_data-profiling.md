# 16 — Data Profiling: Schema, Shape & Volume (نظام البيانات · شكل البيانات · حجم البيانات)

> `15_data-quality.md` asks: **are the values true?**
> This file asks: **is the structure right, does the data look like it should, and does the system survive real volume?**
> Run this BEFORE 15 on any new dataset, migration, or integration — you can't validate values against a schema you haven't verified.

---

## The 3 Layers

| Layer | Arabic | Question | Failure Example |
|-------|--------|----------|-----------------|
| **Schema** (نظام البيانات) | المخطط والعقد | Does the actual structure match the declared contract? | API returns `price: string`, client expects `number` — silent `NaN` in totals |
| **Shape** (شكل البيانات) | التوزيع والتنميط | Does the data statistically look like production data should? | 94% of `country` = "XX" test seed leaked to prod |
| **Volume** (حجم البيانات) | الحجم والنمو | Does everything still work at 10×–100× current scale? | Dashboard query fine at 1K orders, 14s timeout at 500K |

---

## Layer 1 — Schema Validation (نظام البيانات)

### What to verify

- [ ] **Declared vs actual:** DB schema = ORM/validator schema = API contract = client types. Four sources, one truth.
- [ ] **Schema drift:** compare current schema against last-known-good snapshot. Any undocumented column/type change = finding.
- [ ] **Contract enforcement at boundaries:** every external input validated at the edge (Zod / Convex `v.*` / Prisma types), never trusted raw.
- [ ] **Nullability honesty:** columns declared `NOT NULL` in intent but nullable in DB = S2.
- [ ] **Enum closure:** values in data ⊆ values in enum definition. Orphan values = drift.
- [ ] **Foreign key integrity:** in systems without FK enforcement (Convex), orphaned references after deletes.

### Snippets

```sql
-- Postgres/Supabase: snapshot the live schema for drift diffing
SELECT table_name, column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_schema = 'public'
ORDER BY table_name, ordinal_position;
-- Save output → diff against previous snapshot on every release.

-- Enum closure: values outside the allowed set
SELECT DISTINCT status FROM orders
WHERE status NOT IN ('draft','pending','paid','cancelled','refunded');

-- Orphaned references (no FK / soft-FK systems)
SELECT o.id FROM orders o
LEFT JOIN merchants m ON m.id = o.merchant_id
WHERE m.id IS NULL;
```

```ts
// Convex: schema IS the contract — but verify legacy docs comply
// Docs written before a schema change can violate the current validator.
const docs = await ctx.db.query("orders").collect();
const bad = docs.filter(d => typeof d.total !== "number" || d.total < 0);

// API contract test (pairs with 12_api-contract-testing.md):
// response shape validated with Zod, not eyeballed
const OrderSchema = z.object({
  id: z.string(),
  total: z.number().nonnegative(),
  currency: z.enum(["ILS","USD","AED"]),
  createdAt: z.string().datetime(),
});
OrderSchema.parse(await res.json()); // throws = contract violation = S2
```

**Severity:** type mismatch on money/quantity fields = **S1**. Undocumented drift on any user-facing field = S2. Cosmetic (comment/default drift) = S4.

---

## Layer 2 — Shape Profiling (شكل البيانات)

Profile every table/collection in scope. The profile IS the evidence — attach it to the report.

### Standard Profile (per table)

```markdown
## Profile — [table] | Rows: [N] | Profiled: [YYYY-MM-DD]
| Column | Type | Null % | Distinct | Min | Max | Top value (freq %) | Flag |
|--------|------|--------|----------|-----|-----|--------------------|------|
| total | numeric | 0% | 4,812 | 0.5 | 92,400 | — | ✅ |
| country | text | 2% | 3 | — | — | "XX" (94%) | 🔴 seed leak |
| created_at | timestamptz | 0% | — | 2024-01-01 | 2026-07-06 | — | ✅ |
```

### Shape red flags (auto-findings)

| Pattern | Meaning | Severity |
|---------|---------|----------|
| One value > 90% frequency in a business column | Seed/default leak, dead feature, or broken write path | S2 |
| Cardinality ≈ row count on a category column | Free-text where enum expected | S3 |
| Cardinality = 1 on a required column | Column never actually used | S3 |
| Numeric min = 0 spike / max = type boundary (2147483647) | Overflow or default-zero bug | S2 |
| Timestamp cluster at a single second | Bulk backfill masquerading as organic data | S3 (S1 if it feeds analytics revenue) |
| Sudden distribution shift vs last profile | Upstream change nobody announced | S2 |
| Arabic columns: mixed Arabic-Indic/Western digits, unnormalized ة/ه أ/ا | Search & uniqueness silently broken | S2 |

### Profiling commands

```sql
-- One-shot column profile
SELECT
  COUNT(*) AS rows,
  COUNT(*) FILTER (WHERE country IS NULL)::float / COUNT(*) AS null_rate,
  COUNT(DISTINCT country) AS distinct_vals,
  MODE() WITHIN GROUP (ORDER BY country) AS top_value
FROM users;

-- Distribution shift vs baseline (store baseline per release)
SELECT country, COUNT(*)::float / SUM(COUNT(*)) OVER () AS pct
FROM users GROUP BY country ORDER BY pct DESC LIMIT 10;
```

**Rule:** profiles are versioned artifacts. Store the profile per release; the *diff between profiles* is where regressions hide.

---

## Layer 3 — Volume & Scale (حجم البيانات)

### Volume audit (current state)

- [ ] Row counts per table + 30/90-day growth rate → project 6-month size.
- [ ] Largest rows (bloated JSON columns, base64 blobs in text fields).
- [ ] Index coverage on every column used in WHERE/ORDER BY of hot queries.
- [ ] Table scan detection: `EXPLAIN ANALYZE` on the 5 hottest queries — any `Seq Scan` on a table > 10K rows = finding.

```sql
-- Size + growth per table
SELECT relname, n_live_tup,
       pg_size_pretty(pg_total_relation_size(relid)) AS total_size
FROM pg_stat_user_tables ORDER BY n_live_tup DESC;

-- Growth rate (needs created_at)
SELECT date_trunc('week', created_at) AS wk, COUNT(*)
FROM orders GROUP BY 1 ORDER BY 1 DESC LIMIT 12;
```

### Scale test (10× rule)

Never certify a release tested only at current volume. Seed a staging environment at **10× current production rows** (100× for pre-launch products with growth ambitions), then:

- [ ] Re-run the 5 hottest queries → all under threshold (list per query; default: p95 < 300ms API, < 1s dashboard aggregate)
- [ ] Pagination behavior at page 1, page N/2, last page (OFFSET death test — prefer keyset/cursor)
- [ ] Export/report generation completes without timeout at 10×
- [ ] Realtime layers (Convex subscriptions / Supabase realtime) — subscription fan-out at 10× connected clients
- [ ] Background jobs / crons finish inside their schedule window at 10×

**Seeding:** generate synthetic data that matches the Layer-2 profile (same distributions, same null rates, same Arabic/Latin mix) — uniform random data hides real hotspots.

| Finding | Severity |
|---------|----------|
| Hot query > 5× threshold at 10× volume | S1 (will fail in months, deterministic) |
| Missing index on hot path | S2 |
| OFFSET pagination on unbounded table | S2 |
| Export timeout at 10× | S2 |
| Cron overrun at 10× | S3 |

---

## Report Template

```markdown
# Data Profiling Report — [Product / Dataset]
**Prepared by:** [OWNER_NAME] | **Date:** [YYYY-MM-DD]

## Verdict per layer
| Layer | Status | Blockers |
|-------|--------|----------|
| Schema (نظام) | 🟢/🟡/🔴 | [drift count / contract violations] |
| Shape (شكل) | 🟢/🟡/🔴 | [red-flag count] |
| Volume (حجم) | 🟢/🟡/🔴 | [failed scale checks] |

## Schema Drift Diff
[before → after, per changed column]

## Table Profiles
[Layer-2 profile tables, one per table in scope]

## Scale Test Results (at [10×] = [N] rows)
| Query/Flow | Baseline | At scale | Threshold | Pass |
|-----------|----------|----------|-----------|------|

## Findings
### DP-[XXX] — [Title] (S1–S4)
**Layer:** Schema/Shape/Volume | **Evidence:** [query + numbers]
**Fix:** [index / validator / migration / cursor pagination]

## Trace
Linked REQs: [RTM] | Feeds: 15_data-quality (value checks next) + 06_release-gate
```

Push per `10_channel-integration.md`.

---

## Sequencing

```
16 (structure)  →  15 (values)  →  09 (performance)  →  06 (gate)
```

In Super Power missions (`13`): schema/shape/volume runs immediately after `smart-plan` when any data-touching REQ exists — it defines what "correct data" even means before 15 validates it.
