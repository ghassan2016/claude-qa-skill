# 15 — Data Quality & Accuracy (دقة جودة البيانات)

> Functional tests prove the UI works. Data quality tests prove the **numbers are true**.
> A dashboard that renders perfectly but shows a wrong total is an S1, not a cosmetic bug.
> Prerequisite: run `16_data-profiling.md` (schema/shape/volume) first on any new dataset — value checks assume a verified structure.

---

## The 6 Dimensions

Every data quality assessment scores all six. No cherry-picking.

| # | Dimension | Question | Example Failure |
|---|-----------|----------|-----------------|
| 1 | **Accuracy** | Does the stored value match reality/source? | Invoice total ≠ sum of line items |
| 2 | **Completeness** | Are required fields populated? | 12% of orders missing `merchant_id` |
| 3 | **Consistency** | Same fact identical everywhere? | Dashboard says 340 orders, export says 337 |
| 4 | **Validity** | Values within allowed format/range? | Negative quantity, phone without country code, date `0000-00-00` |
| 5 | **Uniqueness** | No unintended duplicates? | Same email → two accounts, double-submitted order |
| 6 | **Timeliness** | Data fresh within SLA? | Realtime dashboard lagging 40 min behind writes |

### Data Quality Score

```
DQ Score = Σ(dimension score 0–100 × weight) / Σ(weights)
Default weights: Accuracy 30, Consistency 20, Completeness 15, Validity 15, Uniqueness 10, Timeliness 10
```

| Score | Verdict |
|-------|---------|
| ≥ 95 | 🟢 Release-grade |
| 85–94 | 🟡 Ship with logged debt (P2 items) |
| < 85 | 🔴 Blocks release — feeds NO-GO in `06` |

### Severity Mapping

| Finding | Severity |
|---------|----------|
| Money/quantity math wrong (accuracy) | **S1** |
| Cross-tenant data visible (uniqueness/boundary) | **S1 + Security Critical** |
| Aggregate ≠ detail (consistency) in user-facing reports | S2 |
| Required-field nulls > 5% | S2 |
| Format violations, stale non-critical data | S3 |
| Cosmetic formatting (trailing spaces, casing) | S4 |

---

## Check Library

### SQL / Supabase (Postgres)

```sql
-- ACCURACY: aggregate vs detail (the #1 money bug)
SELECT i.id, i.total, SUM(li.qty * li.unit_price) AS computed
FROM invoices i JOIN line_items li ON li.invoice_id = i.id
GROUP BY i.id, i.total
HAVING i.total <> SUM(li.qty * li.unit_price);

-- COMPLETENESS: null rate per required column
SELECT COUNT(*) FILTER (WHERE merchant_id IS NULL)::float / COUNT(*) AS null_rate
FROM orders;

-- UNIQUENESS: logical duplicates (same user, same amount, <5s apart = double submit)
SELECT user_id, amount, COUNT(*) FROM payments
GROUP BY user_id, amount, date_trunc('minute', created_at)
HAVING COUNT(*) > 1;

-- VALIDITY: out-of-range
SELECT id FROM line_items WHERE qty <= 0 OR unit_price < 0;

-- CONSISTENCY: two sources of the same fact
SELECT (SELECT COUNT(*) FROM orders WHERE status='paid')
     - (SELECT COUNT(*) FROM payments WHERE status='succeeded') AS drift;

-- TIMELINESS: freshness lag
SELECT now() - MAX(updated_at) AS staleness FROM analytics_daily;
```

**Supabase-specific:** run every check twice — once as `service_role`, once as an authenticated user. A row-count difference is either correct RLS or a **cross-tenant leak (S1)**. Prove which.

### Convex

```ts
// CONSISTENCY: denormalized counter vs actual documents
const posts = await ctx.db.query("posts")
  .withIndex("by_author", q => q.eq("authorId", userId)).collect();
const user = await ctx.db.get(userId);
assert(user.postCount === posts.length, "Counter drift");

// ACCURACY: every mutation that writes money MUST assert invariants
// (aligns with domainEvents assertion discipline)
assert(order.total === lineItems.reduce((s, li) => s + li.qty * li.price, 0));
```

**Convex-specific checks:** denormalized fields vs source-of-truth, orphaned references after deletes (no FK enforcement), optimistic-update rollback leaving stale client state.

### Arabic / RTL Data Validity (first-class, per skill principle #9)

- Arabic-Indic digits (٠١٢٣) vs Western digits stored inconsistently → breaks numeric sorting & search.
- Mixed-direction strings (Arabic name + Latin SKU) corrupting exports/PDFs.
- Normalization: ة/ه ، أ/ا ، ى/ي variants creating false duplicates in search and uniqueness checks.
- Currency/date locale: same value must render identically in dashboard, export, and PDF.

---

## Migration & Pipeline Testing

Any schema migration or ETL gets this checklist:

- [ ] Row count: source = destination (± documented exclusions)
- [ ] Checksums on money columns: `SUM(amount)` before = after
- [ ] Null map unchanged for required fields
- [ ] Sample audit: 20 random rows compared field-by-field
- [ ] Rollback tested — not just written
- [ ] Post-migration: full 6-dimension pass on affected tables

---

## Report Template

```markdown
# Data Quality Report — [Product / Dataset]
**Prepared by:** [OWNER_NAME] | **Date:** [YYYY-MM-DD] | **Scope:** [tables/flows]

## DQ Score: [XX]/100 — 🟢/🟡/🔴

| Dimension | Score | Checks Run | Failed | Worst Finding |
|-----------|-------|------------|--------|---------------|
| Accuracy | | | | |
| Completeness | | | | |
| Consistency | | | | |
| Validity | | | | |
| Uniqueness | | | | |
| Timeliness | | | | |

## Findings
### DQ-[XXX] — [Title]  (Severity: S1–S4)
**Dimension:** [which] | **Evidence:** [query + result count]
**Impact:** [business terms — "totals overstated by 3.2%"]
**Fix:** [remediation + prevention: constraint / assertion / index]

## Trace
Linked REQs: [from RTM in 14] | Feeds release gate: YES
```

Push per `10_channel-integration.md`. DQ Score < 85 → include in release-gate NO-GO justification.
