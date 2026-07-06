# 14 — Smart Planning & Traceability

> Intelligent, risk-based planning + Requirements Traceability Matrix (RTM).
> This is how the skill guarantees the **correct path**: every requirement is traced to a test, every test to a result, every failure to a bug, every bug to a release decision. Nothing ships untracked.

---

## Part 1 — Risk-Based Planning (التخطيط الذكي)

Never plan by listing features top-to-bottom. Plan by **risk = impact × likelihood**.

### Risk Scoring

| Factor | 1 (Low) | 2 (Med) | 3 (High) |
|--------|---------|---------|----------|
| **Impact** | Cosmetic, internal tool | Feature degraded, revenue-adjacent | Money, auth, data loss, legal |
| **Likelihood** | Stable code, no changes | Modified this sprint | New code, new integration, complex logic |

**Risk score = Impact × Likelihood (1–9)**

| Score | Class | Test Depth |
|-------|-------|-----------|
| 6–9 | 🔴 R1 | Full: functional + negative + edge + security + data quality + regression |
| 3–4 | 🟡 R2 | Functional + key negatives + smoke regression |
| 1–2 | 🟢 R3 | Smoke only |

### Auto-R1 Triggers (always highest risk, no scoring needed)
- Payment / wallet / escrow flows
- Auth, session, RBAC/permissions
- Data migrations & schema changes
- Anything writing to production data
- Multi-tenant boundaries (one merchant seeing another's data)
- Arabic/RTL rendering in user-facing money or legal text

### Smart Plan Template

```markdown
# Smart Test Plan — [Feature / Release]
**Prepared by:** [OWNER_NAME] | **Date:** [YYYY-MM-DD]

## Risk Register
| ID | Area | Impact | Likelihood | Score | Class | Depth |
|----|------|--------|-----------|-------|-------|-------|
| R-01 | Checkout flow | 3 | 3 | 9 | 🔴 R1 | Full |
| R-02 | Profile edit | 2 | 2 | 4 | 🟡 R2 | Standard |

## Effort Allocation (rule: 60/30/10)
- 60% of test effort → R1 areas
- 30% → R2 areas
- 10% → R3 smoke

## Phase Plan
| Phase | Scope | Entry Criteria | Exit Criteria | Est. |
|-------|-------|----------------|---------------|------|
| 1 | R1 functional | Deployed to staging + data seeded | 100% R1 cases run, 0 open S1 | [X h] |
| 2 | R2 + data quality | Phase 1 exit met | ≥95% pass, 0 open S1/S2 | [X h] |
| 3 | Regression + gate | Phase 2 exit met | GO/NO-GO issued | [X h] |

## Assumptions & Out of Scope
- [Explicit — anything not listed here is NOT covered]
```

**Planning rules:**
1. Entry/exit criteria are contracts — a phase without met entry criteria does not start.
2. If scope changes mid-plan → log a Drift Event (Part 3), re-score risk, update plan. Never silently absorb scope.
3. Every plan line must be traceable — no test activity without an RTM row (Part 2).

---

## Part 2 — Requirements Traceability Matrix (تتبع المسار الصحيح)

The RTM is the spine of the engagement. It answers, at any moment:
**"Is every requirement tested, and did it pass?"**

### RTM Template

```markdown
# RTM — [Feature / Release]
| REQ ID | Requirement (one line) | Risk | Test Case(s) | Last Run | Result | Bug(s) | Release Status |
|--------|------------------------|------|--------------|----------|--------|--------|----------------|
| REQ-001 | Merchant can create invoice | 🔴 R1 | TC-101, TC-102, TC-103 | 2026-07-06 | ✅ Pass | — | Cleared |
| REQ-002 | Invoice totals match line items | 🔴 R1 | TC-104 | 2026-07-06 | ❌ Fail | BUG-017 (S2) | 🚫 Blocked |
| REQ-003 | RTL layout on invoice PDF | 🟡 R2 | TC-105 | — | ⏳ Not run | — | Pending |
```

### RTM Rules (non-negotiable)

1. **Full forward trace:** every REQ → ≥1 TC. A requirement with zero test cases is a **coverage gap** — report it, never hide it.
2. **Full backward trace:** every TC → exactly one or more REQs. A test case tracing to nothing is scope creep — flag it.
3. **Every FAIL → a bug ID.** No orphan failures.
4. **Every bug → back to its REQ.** When the bug closes, the TC re-runs, the RTM updates. A bug is not "done" until its RTM row is green.
5. **Release status column feeds the release gate (06):** GO requires every R1 row = Cleared and zero Blocked rows.

### Coverage Metric

```
RTM Coverage % = (REQs with ≥1 passing TC) / (Total REQs) × 100
```

Report this number in every checkpoint and in the Master Report (13). Target: **100% for R1, ≥90% for R2**.

---

## Part 3 — Path Integrity Tracking (البقاء على المسار الصحيح)

Plans drift. Track the drift instead of pretending it didn't happen.

### Drift Event Log

```markdown
## Drift Log
| # | Date | What changed | Cause | Impact on plan | Action taken |
|---|------|-------------|-------|----------------|--------------|
| D-1 | 07-06 | New payment provider added mid-sprint | Scope add | +6 R1 cases, +1 day | Re-scored risk, plan v1.1 issued |
```

### Checkpoint Ritual (run at every phase boundary)

1. Print current Mission Board / phase table.
2. Print RTM coverage %.
3. Compare: planned cases vs run cases vs passed.
4. Any gap or drift → log Drift Event + state the corrective action **before** starting the next phase.
5. Verdict: `ON-PATH` / `DRIFTED-CORRECTED` / `OFF-PATH (escalate)`.

**OFF-PATH triggers escalation to ESCALATION_LEAD:** >20% of planned R1 cases skipped, or 2+ uncorrected drift events, or exit criteria waived without sign-off.

---

## Integration Points

- Called first by `13_super-power.md` in every mission.
- Produces the Risk Register consumed by `01_test-plans.md` templates.
- RTM "Release Status" column is a direct input to `06_release-gate.md`.
- Data-critical REQs (money, totals, migrations) automatically add a `15_data-quality.md` phase.
