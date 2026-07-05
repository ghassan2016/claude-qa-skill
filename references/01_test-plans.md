# 01 — Test Plans & Test Cases

---

## Test Plan Template

```markdown
# Test Plan — [Feature / Module Name]
**Project:** [Project Name]  
**Version:** [v1.0]  
**Prepared by:** [OWNER_NAME]  
**Date:** [YYYY-MM-DD]  
**Sprint / Milestone:** [Sprint X]

---

## 1. Scope
**In Scope:**
- [What IS being tested]

**Out of Scope:**
- [What is NOT being tested and why]

## 2. Objectives
- [ ] Verify [core functionality]
- [ ] Validate [edge cases]
- [ ] Confirm [integration points]
- [ ] Check [performance baseline]

## 3. Test Environment
| Environment | URL / Device | Notes |
|------------|-------------|-------|
| Staging | https://staging.xxx.com | Seeded data |
| Mobile iOS | iPhone 14 / iOS 17 | Real device |
| Mobile Android | Samsung S23 / Android 14 | Real device |
| Desktop | Chrome (Latest), Safari (Latest), Firefox (Latest) | |

## 4. Test Data Requirements
- [ ] Test accounts (admin / user / guest)
- [ ] Edge case data (empty states, max length, special chars)
- [ ] Arabic/RTL content samples

## 5. Entry Criteria
- [ ] Feature deployed to staging
- [ ] Test data seeded
- [ ] Previous P0/P1 bugs resolved

## 6. Exit Criteria
- [ ] All P0/P1 test cases passed
- [ ] Zero S1/S2 open bugs
- [ ] Test coverage ≥ 80% of defined cases

## 7. Risk & Assumptions
| Risk | Likelihood | Mitigation |
|------|-----------|-----------|
| [Risk] | High/Med/Low | [Plan] |
```

---

## Test Case Template

```markdown
### TC-[XXX] — [Test Case Title]
**Feature:** [Feature name]  
**Priority:** P0 / P1 / P2 / P3  
**Type:** Functional / UX / Security / Regression  

**Preconditions:**
- User is logged in
- [Any setup needed]

**Steps:**
1. Navigate to [URL/Screen]
2. Enter [input data]
3. Click [action]
4. [Continue steps]

**Expected Result:**
[What should happen — specific and measurable]

**Actual Result:**
[ ] Pass — matches expected  
[ ] Fail — [describe deviation]  
[ ] Blocked — [reason]

**Evidence:**
- Screenshot: [link]
- Video: [link]
- Console log: [paste or link]
```

---

## Platform Test Matrix

### Web Browsers
| Browser | Version | Desktop | Mobile Web |
|---------|---------|---------|-----------|
| Chrome | Latest | ✅ Required | ✅ Required |
| Safari | Latest | ✅ Required | ✅ Required (iOS) |
| Firefox | Latest | ✅ Required | ⚠️ Optional |
| Edge | Latest | ⚠️ Optional | — |

### Mobile Devices
| Platform | Version | Priority |
|---------|---------|---------|
| iOS | 16, 17 | P0 |
| Android | 12, 13, 14 | P0 |
| iPad | iPadOS 16+ | P1 |

### RTL / Arabic Testing
- [ ] Layout flips correctly on RTL
- [ ] Text alignment correct in all components
- [ ] Form fields accept Arabic input
- [ ] Error messages display in Arabic
- [ ] Date/number formats correct for Arabic locale

---

## Feature-Type Test Checklists

### Authentication Features
- [ ] Login with valid credentials → success
- [ ] Login with invalid credentials → correct error message
- [ ] Password reset flow end-to-end
- [ ] Token expiry behavior
- [ ] Concurrent session handling
- [ ] Remember me / stay logged in
- [ ] Logout clears all session data
- [ ] Social login (if applicable)
- [ ] MFA flow (if applicable)
- [ ] Brute force lockout (security)

### Form Features
- [ ] All required fields validated
- [ ] Error messages clear and actionable
- [ ] Success state confirmed
- [ ] Empty state handled
- [ ] Max length enforced on all fields
- [ ] Special characters / XSS input handled
- [ ] Arabic input accepted
- [ ] Keyboard tab order logical
- [ ] Form submits once (no double submit)

### List / Table Features
- [ ] Empty state shown when no data
- [ ] Pagination / infinite scroll works
- [ ] Sorting works correctly
- [ ] Filtering returns accurate results
- [ ] Search handles partial match, Arabic, special chars
- [ ] Loading state shown
- [ ] Error state shown on fetch failure

### Payment / Transaction Features
- [ ] Successful payment flow
- [ ] Failed payment shows correct error
- [ ] Retry payment logic
- [ ] Receipt / confirmation generated
- [ ] Transaction logged in system
- [ ] Refund flow (if applicable)
- [ ] Edge: insufficient balance
- [ ] Edge: network interruption mid-payment

### API Endpoints
- [ ] 200 OK with valid input
- [ ] 400 Bad Request with invalid input
- [ ] 401 Unauthorized without token
- [ ] 403 Forbidden with wrong role
- [ ] 404 Not Found for missing resource
- [ ] 429 Rate limit enforced
- [ ] 500 error does not expose stack trace
- [ ] Response schema matches contract
