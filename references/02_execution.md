# 02 — Test Execution & Reporting

---

## Execution Report Template

```markdown
# Execution Report — [Feature / Sprint / Release]
**Project:** [Name]  
**Tester:** [OWNER_NAME]  
**Date:** [YYYY-MM-DD]  
**Build / Version:** [v1.x.x]  
**Environment:** [Staging / Production]

---

## Summary

| Metric | Count |
|--------|-------|
| Total Test Cases | XX |
| Passed | XX |
| Failed | XX |
| Blocked | XX |
| Not Executed | XX |
| **Pass Rate** | **XX%** |

## Overall Status
🔴 FAIL — [X] P0/P1 open issues  
🟡 CONDITIONAL — Minor issues, documented  
🟢 PASS — Ready for release gate  

---

## Findings

### 🔴 Critical / P0
| ID | Title | Severity | Status |
|----|-------|---------|--------|
| BUG-001 | [Title] | S1 | Open |

### 🟠 High / P1
| ID | Title | Severity | Status |
|----|-------|---------|--------|
| BUG-002 | [Title] | S2 | Open |

### 🟡 Medium / P2
| ID | Title | Severity | Status |
|----|-------|---------|--------|
| BUG-003 | [Title] | S3 | Open |

### 🟢 Low / P3
| ID | Title | Severity | Status |
|----|-------|---------|--------|
| BUG-004 | [Title] | S4 | Open |

---

## Test Case Results

| TC ID | Title | Priority | Result | Notes |
|-------|-------|---------|--------|-------|
| TC-001 | [Title] | P0 | ✅ Pass | |
| TC-002 | [Title] | P0 | ❌ Fail | BUG-001 |
| TC-003 | [Title] | P1 | ⚠️ Blocked | Needs test data |

---

## Environment Issues
- [Any infra/env problems that affected testing]

## Recommendations
1. [Action 1]
2. [Action 2]

## Next Steps
- [ ] Dev fixes BUG-001 by [date]
- [ ] Re-test after fix
- [ ] Final sign-off on [date]
```

---

## Execution Discipline Rules

### Before Starting
- [ ] Confirm build version matches expected
- [ ] Verify test environment is stable
- [ ] Test data seeded and verified
- [ ] Previous blocking bugs resolved
- [ ] Test plan reviewed and signed off

### During Execution
- **One test case at a time** — don't rush through flows
- **Screenshot every failure** — no exceptions
- **Record console/network on failure** — DevTools open
- **Note exact steps** — what you clicked, what you typed
- **Don't fix and re-test without noting the fix** — track the state
- **Test edge cases** after happy path passes
- **Retest every previously-fixed bug** (regression)

### Blocking Rules
If any of these occur → **STOP and escalate immediately:**
- Data corruption or loss
- Security vulnerability discovered
- Auth bypass found
- Payment flow broken
- Database errors in production-like env

**Escalation Path:**
1. Stop all testing — document current state
2. Generate Emergency Push payload (see `10_channel-integration.md → Emergency Push`)
3. Push to Discord with `@here` + set Jira to Highest priority
4. Notify `ESCALATION_LEAD` (configured in SKILL.md) directly — DM or call
5. Do NOT attempt to fix or reproduce further without Dev Lead present for S1/Security Critical

### Evidence Standards
| Finding Type | Required Evidence |
|-------------|------------------|
| UI Bug | Screenshot + Browser + Viewport size |
| Functional Bug | Steps + Screenshot + Console log |
| Performance Issue | Lighthouse report + Network waterfall |
| Security Issue | Request/Response capture + Reproduction steps |
| Data Issue | API response + Database state before/after |

---

## Regression Testing Protocol

### When to Run Full Regression
- Before any production release
- After S1/S2 bug fix
- After major refactor
- After dependency upgrade (framework, auth lib, etc.)

### Regression Scope by Change Type
| Change | Regression Scope |
|--------|----------------|
| UI-only change | Visual + affected flows only |
| Auth change | Full auth suite + session tests |
| API change | Full API contract + all dependent UI flows |
| DB migration | Data integrity + all read/write flows |
| Payment change | Full payment suite + receipts + logging |
| Full release | All P0/P1 test cases |

---

## Pass/Fail Criteria

### Release Blocking (must all pass for GO)
- Zero S1 open bugs
- Zero S2 open bugs (or explicit sign-off with workaround)
- Pass rate ≥ 90% on P0 test cases
- Pass rate ≥ 80% on P1 test cases
- All security scan findings S-Critical resolved
- All security scan findings S-High resolved or have approved workaround

### Non-Blocking (can release with tracking)
- S3 bugs — must be logged with P2 priority
- S4 bugs — logged in backlog
- P2/P3 test case failures with documented justification
