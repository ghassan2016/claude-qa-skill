# 03 — Bug Reports

---

## Standard Bug Report Template

```markdown
## BUG-[XXX] — [Clear, Specific Title]

**Project:** [Name]  
**Reporter:** [OWNER_NAME]  
**Date:** [YYYY-MM-DD]  
**Build / Version:** [v1.x.x]  

---

### Classification
| Field | Value |
|-------|-------|
| **Severity** | S1 Critical / S2 High / S3 Medium / S4 Low |
| **Priority** | P0 / P1 / P2 / P3 |
| **Type** | Functional / Visual / Security / Performance / Data |
| **Status** | Open / In Progress / Fixed / Closed / Won't Fix |
| **Assigned To** | [Developer name] |

---

### Environment
| Field | Value |
|-------|-------|
| Platform | Web / iOS / Android / API |
| Browser/Device | Chrome 120 / iPhone 14 iOS 17 |
| OS | macOS 14 / Windows 11 / Android 14 |
| Screen Resolution | 1440×900 / 390×844 |
| Language/Locale | AR-PS / EN-US |
| User Role | Admin / User / Guest |
| Account Used | test@example.com |

---

### Description
[One clear paragraph: what breaks, where it breaks, what the impact is.]

---

### Steps to Reproduce
1. Navigate to [URL/screen]
2. [Action]
3. [Action]
4. [Observe result]

**Reproducibility:** Always / Sometimes (X/5 attempts) / Rare

---

### Expected Result
[What should happen — specific, measurable]

---

### Actual Result
[What actually happens — specific, observable]

---

### Evidence
- **Screenshot:** [link or embedded]
- **Video:** [link]
- **Console Errors:** [paste or link]
- **Network Request:** [paste HAR or relevant call]
- **Error Log:** [paste relevant lines]

---

### Impact Assessment
- **Users Affected:** All / [Role] / [% estimate]
- **Workaround Available:** Yes / No — [if yes, describe]
- **Business Impact:** [Revenue / Trust / Data / Compliance]

---

### Notes / Additional Context
[Any relevant context: related to recent PR, only happens after X action, etc.]
```

---

## Severity Guide — Detailed

### S1 — Critical
**Definition:** System unusable. Data loss. Security breach. Complete failure of core feature.  
**Examples:**
- User cannot log in or is logged in as wrong user
- Payment processed but not recorded
- Data deleted unintentionally
- Personal data exposed to unauthorized users
- App crashes on launch
- Authentication bypass possible

**Action:** Block release immediately. Escalate to dev lead now.

---

### S2 — High
**Definition:** Major feature broken. Significant user impact. Security vulnerability (non-critical).  
**Examples:**
- Create/Edit form doesn't save data
- Notification system completely silent
- Wrong data displayed to user
- File upload fails silently
- Stored XSS possible
- IDOR on resource read (not write)
- Mobile app crashes on specific flow
- Arabic/RTL layout completely broken

**Action:** Must fix before release. If expedited release needed → explicit sign-off required.

---

### S3 — Medium
**Definition:** Feature partially works. Workaround exists. Cosmetic issue with functional impact.  
**Examples:**
- Pagination shows wrong page count
- Sorting works but order is inconsistent
- Error message is unclear or missing
- Loading state missing (causes confusion)
- Minor layout break on specific viewport
- Form validation message in wrong language

**Action:** Fix next sprint. Log with P2 priority.

---

### S4 — Low
**Definition:** Minor visual issue. Typo. Edge case with minimal user impact.  
**Examples:**
- Typo in non-critical UI text
- Icon slightly misaligned
- Color slightly off from design spec
- Animation jank on low-end device
- Console warning (not error)

**Action:** Add to backlog. Fix when touching related area.

---

## Bug Report Quality Rules

### A good bug report MUST have:
- [ ] Exact steps to reproduce (not "I clicked around and it broke")
- [ ] Specific expected vs actual result
- [ ] At least one screenshot or video
- [ ] Environment details (browser, device, account role)
- [ ] Correct severity — don't over or under-classify

### A bad bug report looks like:
```
Title: "Something is wrong with the form"
Description: "When I try to submit it doesn't work"
Steps: "Click submit"
Evidence: none
```

### Red Flags to Escalate Immediately
Any bug that involves:
- User data visible to another user
- Auth token exposed in URL, localStorage, or response
- Admin functionality accessible by regular user
- Payment amount editable client-side
- File upload accepts executable files
- SQL/JS injection possible in any input

---

## Attachments Specification

| Type | Format | How to Capture |
|------|--------|---------------|
| Screenshot | PNG, annotated | Browser screenshot + annotations in Figma/Canva |
| Video | MP4 / GIF (< 10MB) | Loom, Screen Record, FFMPEG GIF |
| Console Log | Plain text | DevTools → Console → Copy all errors |
| Network Request | HAR / JSON | DevTools → Network → Copy as cURL |
| API Response | JSON | Postman / Thunder Client export |
| Crash Log | Plain text | Xcode / Android Studio logcat |
