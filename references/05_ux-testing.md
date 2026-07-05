# 05 — UX Testing

---

## Nielsen's 10 Heuristics — Applied Audit

Use these as the evaluation framework for any interface.

| # | Heuristic | Key Questions |
|---|-----------|--------------|
| H1 | Visibility of System Status | Does the user always know what's happening? (loading, saving, error, success) |
| H2 | Match Between System & Real World | Does the language match user's vocabulary? Are concepts familiar? |
| H3 | User Control & Freedom | Can users undo? Exit easily? Cancel without consequence? |
| H4 | Consistency & Standards | Are similar actions always triggered the same way? |
| H5 | Error Prevention | Does the design prevent errors before they happen? |
| H6 | Recognition Over Recall | Is info visible when needed, not requiring memory? |
| H7 | Flexibility & Efficiency | Can power users use shortcuts? |
| H8 | Aesthetic & Minimalist Design | Is every element earning its place? |
| H9 | Error Recovery | Are error messages helpful and actionable? |
| H10 | Help & Documentation | Is help findable without leaving the flow? |

---

## User Flow Testing

### How to Evaluate Any Flow

1. **Define the goal** — what is the user trying to accomplish?
2. **Walk the happy path** — step by step, note every friction point
3. **Test the failure path** — what happens when input is wrong?
4. **Test the edge cases** — empty state, first use, error recovery
5. **Time the flow** — count clicks and steps to goal
6. **Rate each step** — smooth / minor friction / major friction / blocker

### Key Flows to Always Test

| Flow | Success Condition |
|------|-----------------|
| Onboarding / First use | User reaches first value moment in < 3 minutes |
| Registration / Sign up | Complete in under 5 fields without confusion |
| Login | Under 2 steps to access |
| Core feature (main job-to-be-done) | Achievable without help documentation |
| Error recovery | User understands what went wrong and how to fix it |
| Settings / Account management | User can find and change critical settings |
| Payment / Checkout | Zero ambiguity at every step |
| Log out / Delete account | Accessible without searching |

### Friction Scoring
| Score | Meaning |
|-------|---------|
| 0 — Smooth | No hesitation, no confusion |
| 1 — Minor | Slight pause, recoverable |
| 2 — Friction | Requires re-reading, back-navigation, or re-try |
| 3 — Blocker | User cannot complete task without help |

---

## Onboarding Funnel Audit

- [ ] First screen explains value proposition clearly (< 8 seconds to understand)
- [ ] Required fields minimized (email + password maximum for initial signup)
- [ ] Progress indicator on multi-step flows
- [ ] No dead-ends (every error has a next action)
- [ ] Welcome email / confirmation message exists
- [ ] First session includes guided action toward core value
- [ ] Skip option available for non-essential steps

---

## CTA Hierarchy Validation

- [ ] Primary action is visually dominant on every screen
- [ ] No two competing CTAs at equal visual weight
- [ ] Destructive actions (delete, cancel) are visually differentiated (red / outlined)
- [ ] CTA text is action-oriented (verb + object: "Save Profile" not "Submit")
- [ ] Disabled states visible and explained (tooltip or helper text)
- [ ] Button loading state exists (prevents double-submit frustration)

---

## Form UX Audit

- [ ] Label above field (not placeholder-only — placeholder disappears on focus)
- [ ] Error appears inline below field, not only at top of form
- [ ] Error message says what's wrong AND how to fix it
- [ ] Success validation shown inline (green check) where helpful
- [ ] Required fields marked consistently (asterisk + legend)
- [ ] Field order follows logical mental model (name → email → password)
- [ ] Keyboard type matches field (number keyboard for phone, email keyboard for email)
- [ ] Auto-fill / autocomplete supported where applicable
- [ ] Form submits on Enter key
- [ ] No data lost if user navigates away accidentally (warn or autosave)

---

## Accessibility (a11y) Audit

### WCAG 2.1 AA Checklist

**Color & Contrast**
- [ ] Text contrast ratio ≥ 4.5:1 (normal text)
- [ ] Large text (18pt+) contrast ≥ 3:1
- [ ] Interactive elements not conveyed by color alone
- [ ] Focus indicators visible (not removed with `outline: none`)

**Keyboard Navigation**
- [ ] All interactive elements reachable via Tab key
- [ ] Tab order follows visual/logical reading order
- [ ] Modal/dialog traps focus inside until closed
- [ ] Skip-to-content link present on web

**Screen Reader (ARIA)**
- [ ] Images have descriptive `alt` text (not "image" or empty)
- [ ] Form fields have associated `<label>` elements
- [ ] Error messages linked to field via `aria-describedby`
- [ ] Buttons have accessible names (not icon-only without label)
- [ ] Loading states announced to screen reader (`aria-live`)
- [ ] Language declared on `<html lang="ar">` or `lang="en"`

**RTL / Arabic Specific**
- [ ] `dir="rtl"` set on document or component
- [ ] Layout mirrors correctly (nav, buttons, icons)
- [ ] Text alignment correct (`text-align: start` not hardcoded `right`)
- [ ] Icons that imply direction (arrows, chevrons) mirrored
- [ ] Numbers and dates in correct Arabic/Western format per context

---

## Mobile UX Testing

### Touch Targets
- [ ] All tap targets ≥ 44×44px (iOS HIG / Material minimum)
- [ ] Sufficient spacing between adjacent touch targets (≥ 8px)
- [ ] No tap targets overlapping

### Scroll & Gestures
- [ ] Scrollable areas have visual indicator (fade, scrollbar)
- [ ] Pull-to-refresh works where expected
- [ ] Swipe gestures don't conflict with system gestures
- [ ] Bottom sheet / modal dismisses with swipe-down

### Responsive Breakpoints
| Breakpoint | Width | Test |
|-----------|-------|------|
| Mobile S | 375px | Required |
| Mobile L | 430px | Required |
| Tablet | 768px | Required |
| Desktop | 1280px | Required |
| Wide | 1440px+ | Optional |

### Common Mobile Issues to Check
- [ ] Text not readable without zooming
- [ ] Horizontal scroll occurs (layout overflow)
- [ ] Fixed elements don't cover content
- [ ] Keyboard pushes content up correctly (no hidden fields)
- [ ] Back button behavior is consistent
- [ ] Bottom navigation doesn't conflict with iOS home indicator

---

## Copy Audit

| Check | Standard |
|-------|---------|
| Language consistency | All UI in same language per locale |
| Tone | Consistent (formal vs casual) across entire product |
| Error messages | Actionable — not "An error occurred" |
| Empty states | Instructional — tell user what to do next |
| Button labels | Verb + object — not just "OK" or "Submit" |
| Confirmation dialogs | Specific — "Delete this account?" not "Are you sure?" |
| Success messages | Confirm what happened — "Profile saved" not just "Done" |
| Arabic text | Native Arabic — not literal translation of English phrases |

---

## UX Audit Output Format

```markdown
## UX Review — [Screen / Flow / Feature]
**Reviewer:** [OWNER_NAME]  
**Date:** [YYYY-MM-DD]  

### Summary
[Overall UX health: Good / Needs Work / Significant Issues]

### Findings

#### 🔴 Blockers (prevent task completion)
1. **[Finding]** — H[#]: [Heuristic name]  
   Screen: [Screen name]  
   Issue: [Description]  
   Impact: [User cannot complete X]  
   Fix: [Specific recommendation]

#### 🟠 High Friction (cause confusion or abandonment)
1. **[Finding]** — H[#]

#### 🟡 Minor Friction (improve experience)
1. **[Finding]**

#### ✅ Working Well
- [What to preserve]

### Recommended Priority Queue
1. [Fix 1 — highest impact, lowest effort]
2. [Fix 2]
3. [Fix 3]
```
