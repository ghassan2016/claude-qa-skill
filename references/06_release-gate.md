# 06 — Release Gate: GO / NO-GO

---

## Release Gate Checklist

Run this before every production deployment.  
**All P0 items must be checked before GO.**

---

### Gate 1 — Quality Baseline
| Check | Status | Notes |
|-------|--------|-------|
| Zero S1 open bugs | ☐ | |
| Zero S2 open bugs | ☐ | Exception requires VP sign-off |
| Test pass rate ≥ 90% (P0 cases) | ☐ | |
| Test pass rate ≥ 80% (P1 cases) | ☐ | |
| All P0 bugs from previous release closed | ☐ | |
| Regression suite passed | ☐ | |

### Gate 2 — Security
| Check | Status | Notes |
|-------|--------|-------|
| Zero Critical security findings | ☐ | |
| Zero High security findings (or approved workaround) | ☐ | |
| Auth flows tested (login, logout, token expiry) | ☐ | |
| New API endpoints have authorization checks | ☐ | |
| No hardcoded secrets in released code | ☐ | |
| Environment variables verified in production config | ☐ | |

### Gate 3 — UX / UI
| Check | Status | Notes |
|-------|--------|-------|
| Core user flows tested on mobile + desktop | ☐ | |
| Arabic/RTL display verified (if applicable) | ☐ | |
| Error states visible and actionable | ☐ | |
| Loading states present on all async operations | ☐ | |
| Empty states implemented | ☐ | |
| No broken images or missing icons | ☐ | |

### Gate 4 — Technical
| Check | Status | Notes |
|-------|--------|-------|
| Database migrations tested in staging | ☐ | |
| No breaking API changes without versioning | ☐ | |
| Performance baseline maintained (Lighthouse ≥ 80) | ☐ | |
| No console errors in production build | ☐ | |
| Feature flags configured correctly | ☐ | |
| Third-party integrations tested (payments, email, etc.) | ☐ | |

### Gate 5 — Operations
| Check | Status | Notes |
|-------|--------|-------|
| Rollback plan documented | ☐ | |
| Monitoring / alerts configured | ☐ | |
| Release notes written | ☐ | |
| Support team briefed on changes | ☐ | |
| Deployment window confirmed (low-traffic period) | ☐ | |

---

## Risk Classification

### Release Risk Level

| Level | Criteria | Decision |
|-------|---------|---------|
| 🟢 LOW | All gates pass, no S3+ open | GO — deploy immediately |
| 🟡 MEDIUM | S3 bugs open, P2 test failures, minor UX issues | GO with tracking — log all open items |
| 🟠 HIGH | S2 open with workaround, performance regression, UX blocker | CONDITIONAL GO — explicit sign-off required |
| 🔴 CRITICAL | Any S1, any Critical security, auth broken, payment broken | NO-GO — block release |

---

## GO / NO-GO Decision Template

```markdown
# Release Gate Decision — [Release Name / Version]
**Project:** [Name]  
**Decision Date:** [YYYY-MM-DD]  
**Decided by:** [OWNER_NAME]  
**Release Type:** Hotfix / Feature / Major  

---

## Verdict

### 🟢 GO / 🔴 NO-GO

**Reason:**
[One clear paragraph explaining the decision]

---

## Open Items at Release
| ID | Title | Severity | Owner | ETA |
|----|-------|---------|-------|-----|
| BUG-XXX | [Title] | S3 | [Dev] | [Date] |

## Conditions for GO (if conditional)
- [ ] [Condition 1 must be met]
- [ ] [Condition 2]

## Rollback Trigger
Release will be rolled back if:
- [ ] [Error rate exceeds X%]
- [ ] [Payment failure rate exceeds X%]
- [ ] [Auth failure reports received]
- [ ] [Data corruption detected]

## Rollback Plan
1. [Specific step]
2. [Specific step]
3. [Communication plan]
4. [Data recovery steps if needed]

## Sign-offs
| Role | Name | Decision | Time |
|------|------|---------|------|
| QA Lead | [OWNER_NAME] | Approved / Blocked | |
| Dev Lead | [Name] | Approved | |
| Product | [Name] | Approved | |
```

---

## Post-Release Monitoring Protocol

### First 30 Minutes
- [ ] Monitor error rate dashboard
- [ ] Check payment success rate (if applicable)
- [ ] Verify auth flow works in production
- [ ] Spot-test core user flow end-to-end
- [ ] Watch Sentry / Crashlytics for new errors

### First 2 Hours
- [ ] Check user-facing metrics (conversion, completion rates)
- [ ] Review server error logs
- [ ] Confirm email/notification delivery working
- [ ] Check database performance metrics

### First 24 Hours
- [ ] Review user feedback / support tickets
- [ ] Compare core KPIs vs pre-release baseline
- [ ] Close release gate document with final status

### Rollback Decision Time
- **< 15 minutes** to decide on rollback after critical issue detected
- **Rollback is not failure** — it is quality discipline

---

## Hotfix Release Protocol

When a critical bug is found in production:

1. **Assess severity** — S1: immediate hotfix, S2: scheduled hotfix same day
2. **Reproduce in staging** — confirm fix works before deploy
3. **Minimal scope** — hotfix contains ONLY the critical fix
4. **Abbreviated gate** — Security + Auth + Affected flow minimum
5. **Deploy in off-peak hours** if possible
6. **Post-mortem** within 48 hours for any S1 bug
