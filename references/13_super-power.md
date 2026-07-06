# 13 — Super Power (Orchestrator Mode)

> The command layer. Super Power does not test — it **plans, delegates, tracks, and consolidates**.
> Use for any engagement too big for a single mode: pre-launch audits, multi-sprint QA campaigns, full product certification.

---

## When Super Power Activates

| Signal | Example |
|--------|---------|
| Multi-domain goal | "جهز المنتج للإطلاق", "certify this app", "full QA campaign" |
| Multi-sprint scope | QA plan covering 3+ sprints or 2+ modules |
| Explicit call | "super power", "شغّل السوبر باور", "orchestrate" |
| 3+ modes needed | Any goal that requires 3 or more routing-table modes |

If only ONE mode is needed → do NOT use Super Power. Route directly.

---

## The Orchestration Loop

```
GOAL → MISSION PLAN → EXECUTE PHASE → CHECKPOINT → NEXT PHASE → MASTER REPORT → CHANNEL PUSH
                          ↑                |
                          └── re-plan ←────┘ (if S1/Critical found)
```

1. **Intake** — restate the goal in one sentence. Identify product, stack (auto-detect), deadline, risk appetite.
2. **Mission Plan** — build the Mission Board (template below). Sequence modes with dependencies. Load `14_smart-planning.md` to risk-rank scope.
3. **Execute** — run each phase using its reference file. One phase at a time. Never skip a checkpoint.
4. **Checkpoint** — after each phase: update Mission Board, log findings count by severity, decide CONTINUE / RE-PLAN / HALT.
5. **Master Report** — consolidate all phase findings into one executive report with a single GO/NO-GO.
6. **Push** — send Master Report to OUTPUT_CHANNEL per `10_channel-integration.md`.

**Halt conditions (stop the mission, escalate to ESCALATION_LEAD):**
- Any S1 or Security Critical → escalate first, then decide whether remaining phases still make sense.
- Data corruption or payment/auth broken → immediate halt.

---

## Mission Board Template

Maintain this board across the whole session. Re-print it at every checkpoint.

```markdown
# 🎯 QA Mission Board — [Product / Goal]
**Commander:** [OWNER_NAME] | **Started:** [YYYY-MM-DD] | **Deadline:** [date]
**Goal:** [one sentence]
**Risk appetite:** Strict / Balanced / Fast-ship

## Phases
| # | Phase | Mode | Ref | Status | Findings (S1/S2/S3/S4) | Verdict |
|---|-------|------|-----|--------|------------------------|---------|
| 1 | Risk-based plan | smart-plan | 14 | ✅ Done | — | Plan approved |
| 2 | Functional pass | execute | 02 | 🔄 Active | 0/2/3/1 | — |
| 3 | Data quality | data-quality | 15 | ⏳ Queued | — | — |
| 4 | Security scan | security-scan | 08 | ⏳ Queued | — | — |
| 5 | Performance | performance | 09 | ⏳ Queued | — | — |
| 6 | Release gate | release-gate | 06 | ⏳ Queued | — | GO / NO-GO |

## Blockers
| ID | Severity | Phase | Owner | Status |
|----|----------|-------|-------|--------|

## Path Integrity (from 14_smart-planning.md)
Planned phases: [N] | Completed on-plan: [N] | Drift events: [N] | RTM coverage: [%]
```

**Status legend:** ⏳ Queued · 🔄 Active · ✅ Done · 🚫 Blocked · ⚠️ Re-planned

---

## Phase Sequencing Rules

Default order (adjust per goal, never violate dependencies):

```
smart-plan (14)          ← always first: no execution without a risk-ranked plan
   ↓
code-review (04)         ← if code/PRs are in scope
   ↓
execute (02) + bug-report (03)
   ↓
data-profiling (16)      ← structure first: schema/shape/volume defines what "correct" means
   ↓
data-quality (15)        ← before security: bad data invalidates security test results
   ↓
security-scan (08)
   ↓
ux-review (05) + ui-audit (07)   ← parallel-safe
   ↓
performance (09)
   ↓
e2e (11) / api-contract (12)     ← if automation deliverables requested
   ↓
release-gate (06)        ← always last, consumes ALL phase outputs
```

**Dependency rules:**
- `release-gate` NEVER runs before all in-scope phases complete.
- `data-profiling` (16) runs before `data-quality` (15) — structure before values.
- `data-quality` runs before `security-scan` and `performance` (both depend on realistic data).
- A phase with an open S1 blocks all downstream phases.

---

## Master Report Template

```markdown
# 🏁 QA Mission Report — [Product]
**Prepared by:** [OWNER_NAME] | **Date:** [YYYY-MM-DD] | **Mission duration:** [X days]

## Executive Summary
[3–5 sentences: goal, what was covered, headline findings, final verdict.]

## Verdict: 🟢 GO / 🔴 NO-GO
[One sentence justification. NO-GO must list the exact blockers.]

## Coverage Map
| Domain | Phase | Cases Run | Pass % | Open S1/S2 |
|--------|-------|-----------|--------|------------|

## Findings by Severity
| Severity | Count | Fixed | Open | Deferred |
|----------|-------|-------|------|----------|

## Top 5 Risks Going Forward
1. [Risk + recommendation]

## Traceability Summary
RTM coverage: [%] — [N] requirements fully traced, [N] gaps (see RTM in 14).

## Data Quality Score
[Score /100 from 15_data-quality.md — one line per failing dimension.]

## Deferred Items (P2/P3)
| ID | Item | Target |
|----|------|--------|
```

Then push per `10_channel-integration.md`. In `all` mode: full report → Jira, executive summary → Discord/WhatsApp.

---

## Anti-Patterns (Do NOT)

- ❌ Run all phases in one giant response — one phase per turn, checkpoint between.
- ❌ Ask the user to sequence phases — that is Super Power's job. State the plan, proceed.
- ❌ Produce a GO verdict with any open S1/S2 or Security Critical/High.
- ❌ Lose the Mission Board — re-print it at every checkpoint, always current.
- ❌ Skip `smart-plan` — an unplanned mission is a checklist, not a mission.
