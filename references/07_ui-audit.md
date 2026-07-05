# 07 — UI Audit: Visual Consistency & Design Quality

---

## Visual Consistency Audit

### Design Token Usage
- [ ] Colors use only tokens from the design system (no hardcoded hex values in components)
- [ ] Spacing follows the defined scale (4px / 8px / 12px / 16px / 24px / 32px / 48px / 64px)
- [ ] Typography uses defined scale (no arbitrary `font-size` values)
- [ ] Border radius consistent (don't mix `4px` and `8px` randomly)
- [ ] Shadow/elevation levels consistent
- [ ] Transition durations follow system (100ms / 200ms / 300ms)

### Component State Coverage
Every interactive component must have all of these:

| Component Type | Required States |
|---------------|----------------|
| Button | Default · Hover · Active · Focus · Disabled · Loading |
| Input / Field | Default · Focus · Filled · Error · Disabled · Read-only |
| Link | Default · Hover · Visited · Focus |
| Checkbox / Radio | Unchecked · Checked · Indeterminate · Disabled |
| Toggle | Off · On · Disabled |
| Card | Default · Hover (if interactive) · Selected · Error |
| Dropdown | Closed · Open · Selected · Disabled |

- [ ] All states verified in implementation vs Figma
- [ ] Focus ring visible on all interactive elements (keyboard nav)
- [ ] Disabled state not just greyed — cursor changed to `not-allowed`
- [ ] Loading state prevents interaction (button disabled while loading)

### Icon & Illustration Consistency
- [ ] Single icon library used throughout (no mixing icon sets)
- [ ] Icon sizes follow defined scale (16px / 20px / 24px / 32px)
- [ ] Icon color uses design token (not hardcoded)
- [ ] Directional icons (arrows, chevrons) mirrored in RTL
- [ ] Illustrations share consistent style, color palette, and line weight

### Loading States
- [ ] Every async operation shows a loading indicator
- [ ] Skeleton screens used for content-heavy pages (not spinner-only)
- [ ] Skeleton matches the layout of the loaded content
- [ ] Loading state prevents user input that would cause duplicate requests

### Empty States
- [ ] Every list/table has a designed empty state
- [ ] Empty state explains why it's empty AND what to do next
- [ ] Empty state illustration/icon consistent with design system
- [ ] First-time user experience differentiated from "no results" state

---

## Figma-to-Dev Gap Analysis

### Pixel Tolerance Standards
| Element Type | Acceptable Deviation |
|-------------|-------------------|
| Spacing / Padding | ±4px |
| Font size | 0px (must be exact) |
| Border radius | ±2px |
| Color | 0px (must match token exactly) |
| Component dimensions | ±8px |
| Icon size | ±2px |

### Gap Analysis Checklist
- [ ] Font family matches Figma exactly
- [ ] Font weights correct (400/500/600/700 — not approximated)
- [ ] Letter-spacing matches (especially for headings)
- [ ] Line-height matches (critical for Arabic text)
- [ ] Padding/margin matches on all 4 sides
- [ ] Component dimensions within tolerance
- [ ] Colors match design tokens exactly (no "close enough")
- [ ] Border styles match (solid/dashed, width, color)
- [ ] Shadows match (x, y, blur, spread, color, opacity)
- [ ] All component states implemented (not just default)

### Figma Handoff Review Output
```markdown
## Figma-to-Dev Gap Report — [Screen/Component]

### 🔴 Critical Deviations (must fix)
| Element | Figma | Implemented | Impact |
|---------|-------|-------------|--------|
| [Element] | [spec] | [actual] | [visual/brand] |

### 🟡 Minor Deviations (fix in polish pass)
| Element | Figma | Implemented | Note |
|---------|-------|-------------|------|

### ✅ Matches Spec
[List components that are pixel-perfect]
```

---

## Cross-Platform Visual Testing

### Browser Matrix

| Browser | Version | Test Type | Priority |
|---------|---------|-----------|---------|
| Chrome | Latest | Full audit | P0 |
| Safari | Latest | Full audit (especially CSS quirks) | P0 |
| Firefox | Latest | Spot check | P1 |
| Edge | Latest | Spot check | P2 |
| iOS Safari | Latest | Mobile full audit | P0 |
| Chrome Android | Latest | Mobile full audit | P0 |
| Samsung Internet | Latest | Spot check | P2 |

### Common Cross-Browser Issues to Check
- [ ] CSS Grid / Flexbox behavior consistent
- [ ] `gap` property on Flex (Safari older versions issue)
- [ ] Custom fonts loaded on all browsers
- [ ] Form styling (inputs look different on iOS Safari)
- [ ] Scrollbar styling (hidden vs visible across OS)
- [ ] Animations perform on lower-end hardware
- [ ] `position: sticky` works in all layouts
- [ ] Backdrop filters supported (blur effects)

### Dark Mode Testing (if supported)
- [ ] All text readable in dark mode
- [ ] Images have dark-mode-appropriate backgrounds
- [ ] Icons visible in both modes
- [ ] System dark mode preference respected
- [ ] No hardcoded colors that break dark mode

### RTL Layout Testing
- [ ] Complete layout mirror in RTL mode
- [ ] Navigation items flipped correctly
- [ ] Icons with directional meaning flipped (→ becomes ←)
- [ ] Form field labels on correct side
- [ ] Table column order correct
- [ ] Padding/margin asymmetry handled with logical properties (`margin-inline-start`)
- [ ] Numbers in Arabic-Indic or Western Arabic per user expectation
- [ ] Text alignment correct (`start` not `left`)
- [ ] Arabic text renders with correct font (not fallback)
- [ ] Line heights adequate for Arabic (Arabic needs more than Latin)

---

## Typography Audit

- [ ] Max 3 font families in entire product
- [ ] Heading hierarchy clear (H1 > H2 > H3 visually obvious)
- [ ] Body text readable (min 14px mobile, 16px desktop)
- [ ] Line length optimal (45-75 characters per line)
- [ ] Line height for Arabic ≥ 1.8 (Arabic needs more than 1.5)
- [ ] No all-caps Arabic text (reduces readability)
- [ ] Long text not in light weight (< 400) on dark backgrounds

---

## UI Audit Output Format

```markdown
## UI Audit — [Screen / Component / Feature]
**Reviewer:** [OWNER_NAME]  
**Date:** [YYYY-MM-DD]  
**Compared against:** Figma file [link] / Design system [version]

### Summary
Visual health: Excellent / Good / Needs Rework / Major Issues

---

### 🔴 Critical — Breaks Brand or Usability
1. **[Issue]** — [Component/Screen]  
   Expected: [Figma spec]  
   Actual: [What's in the build]  
   Fix: [Specific correction]

### 🟠 High — Visible Deviation
1. **[Issue]**

### 🟡 Polish — Minor (fix in next pass)
1. **[Issue]**

### ✅ Matches Design
[List correctly implemented components/screens]

### Overall Verdict
🔴 Rework needed / 🟡 Minor fixes / 🟢 Approved for release
```
