# 09 — Performance Testing

---

## Core Web Vitals — Targets & Thresholds

These are Google's official thresholds. Failing them affects SEO ranking.

| Metric | Good | Needs Improvement | Poor | What it measures |
|--------|------|------------------|------|-----------------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | 2.5–4.0s | > 4.0s | Loading — main content visible |
| **INP** (Interaction to Next Paint) | ≤ 200ms | 200–500ms | > 500ms | Interactivity — response to user input |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | 0.1–0.25 | > 0.25 | Visual stability — no content jumping |

> **Note:** FID (First Input Delay) was replaced by INP as a Core Web Vital in March 2024.

---

## Performance Testing Tools

| Tool | What it measures | How to run |
|------|-----------------|-----------|
| **Lighthouse** | Full audit (Performance, A11y, SEO, Best Practices) | Chrome DevTools → Lighthouse tab |
| **WebPageTest** | Real-browser filmstrip, TTFB, waterfall | webpagetest.org |
| **Chrome DevTools Performance** | JS execution, paint, layout, scripting timeline | DevTools → Performance tab → Record |
| **Chrome DevTools Coverage** | Unused JS/CSS by file | DevTools → Coverage (Cmd+Shift+P) |
| **Bundle Analyzer** (Next.js) | Bundle size per route | `ANALYZE=true next build` |
| **React DevTools Profiler** | Component render time, commit count | React DevTools browser extension |
| **Supabase Dashboard** | Query execution time, slow queries | Supabase → Database → Query Performance |

---

## Lighthouse Audit Checklist

Run Lighthouse in **Incognito mode** (no extensions, no cached state):

### Target Scores
| Category | Minimum | Target |
|----------|---------|--------|
| Performance | 80 | 90+ |
| Accessibility | 90 | 100 |
| Best Practices | 95 | 100 |
| SEO | 90 | 100 |

### Common Failure Causes to Check

**LCP Issues**
- [ ] Hero image not preloaded (`<link rel="preload">` or `priority` on `next/image`)
- [ ] LCP element is a background CSS image (not preloadable by browser)
- [ ] Render-blocking resources (CSS, fonts) delaying first paint
- [ ] Server response time (TTFB) > 600ms — check DB query, cold start, or CDN miss
- [ ] No CDN for static assets — serving from origin server

**INP Issues**
- [ ] Long Tasks > 50ms blocking the main thread
- [ ] Large React re-renders on user interaction (unnecessary state updates)
- [ ] Synchronous localStorage access on interaction
- [ ] Third-party scripts (analytics, chat widget) blocking interaction response
- [ ] `useEffect` running expensive computations on every render

**CLS Issues**
- [ ] Images without explicit `width` and `height` (or `aspect-ratio`)
- [ ] Ads or embeds without reserved space
- [ ] Fonts causing layout shift (FOUT/FOIT) — use `font-display: optional` or `swap`
- [ ] Dynamic content injected above existing content
- [ ] Skeleton screens not matching real content dimensions

---

## Bundle Size Analysis

### Targets

| Bundle Type | Warning | Critical |
|-------------|---------|---------|
| Initial JS (First Load) | > 300KB gzipped | > 500KB gzipped |
| Page-specific JS | > 100KB gzipped | > 200KB gzipped |
| CSS | > 50KB gzipped | > 100KB gzipped |

### Checks
- [ ] No large libraries imported fully when tree-shakeable (e.g., `import _ from 'lodash'` → `import debounce from 'lodash/debounce'`)
- [ ] Moment.js used? → Replace with `date-fns` or native `Intl`
- [ ] Icons: importing entire icon pack vs individual icons?
- [ ] Dynamic `import()` used for heavy page-specific code
- [ ] No duplicate dependencies (different versions of same library)
- [ ] `next/dynamic` with `ssr: false` for browser-only heavy libraries

### How to analyze Next.js bundle
```bash
ANALYZE=true next build
# Opens bundle-analyzer in browser — look for:
# - Large modules in the initial load (First Load JS)
# - Duplicate packages (same lib, multiple versions)
# - Unexpected large dependencies
```

---

## Backend / API Performance

### Response Time Targets

| Endpoint Type | Target | Warning |
|---------------|--------|---------|
| Read (simple query) | < 100ms | > 300ms |
| Read (complex/join) | < 300ms | > 600ms |
| Write (single record) | < 200ms | > 500ms |
| Write (batch) | < 1s | > 2s |
| File upload (per MB) | < 500ms | > 1s |

### Database Query Review
- [ ] `EXPLAIN ANALYZE` run on slow queries (> 300ms)
- [ ] Missing indexes on frequently filtered/joined columns
- [ ] N+1 query pattern — checking if queries multiply with data size
- [ ] Pagination implemented — no queries returning unbounded result sets
- [ ] Connection pool configured correctly (especially for serverless)
- [ ] Read replicas used for analytics/reporting queries

### Supabase-Specific Performance
- [ ] Indexes on foreign key columns and commonly-filtered columns
- [ ] Avoid `SELECT *` on large tables — specify columns
- [ ] Use `count: 'estimated'` instead of `count: 'exact'` on large tables
- [ ] Realtime subscriptions scoped to specific rows (not full table)
- [ ] Edge Functions used instead of client-side sequences of API calls

### Convex-Specific Performance
- [ ] Indexes defined with `.index()` for queried fields
- [ ] Pagination used for list queries (`paginate()`)
- [ ] Heavy computations in `action` (not `query`)
- [ ] `ctx.storage` reads not in hot query paths

---

## Mobile Performance (Flutter)

### Frame Rate Targets
| Mode | Target |
|------|--------|
| Normal UI | 60 FPS |
| Animations | 60 FPS (no jank) |
| Scrolling lists | 60 FPS stable |

### Checks
- [ ] `flutter run --profile` — check for jank in DevTools timeline
- [ ] `const` widgets used to prevent unnecessary rebuilds
- [ ] `ListView.builder` for all lists (not `Column` + `.map()`)
- [ ] Images compressed and cached (`cached_network_image`)
- [ ] No synchronous file I/O on UI thread
- [ ] `compute()` used for CPU-intensive work (JSON parsing, compression)

---

## Performance Testing Procedure

### Step 1 — Establish Baseline
1. Run Lighthouse 3× in incognito → record median scores
2. Record current bundle size (`next build` output or Flutter `--analyze`)
3. Identify the top 3 Lighthouse opportunities

### Step 2 — Identify Bottlenecks
1. DevTools Performance tab → record page load → look for Long Tasks (red bar)
2. Network tab → identify largest assets, slowest requests
3. Coverage tab → note unused JS/CSS %
4. Check server TTFB in Network tab (first byte time)

### Step 3 — Profile Interactions
1. Open DevTools Performance tab
2. Click Record → perform the interaction (click, input, route change)
3. Stop → inspect for Long Tasks, script execution time
4. React DevTools Profiler → identify unnecessary re-renders

### Step 4 — Verify Improvements
Run Lighthouse again after fixes — compare against baseline.

---

## Performance Report Output Format

```markdown
## Performance Assessment — [Page / Feature / App]
**Assessor:** [OWNER_NAME]
**Date:** [YYYY-MM-DD]
**Tool:** Lighthouse / WebPageTest / DevTools
**Environment:** Staging / Production
**Device profile:** Mobile (slow 4G) / Desktop

---

### Core Web Vitals

| Metric | Measured | Target | Status |
|--------|---------|--------|--------|
| LCP | X.Xs | ≤ 2.5s | ✅/⚠️/❌ |
| INP | XXms | ≤ 200ms | ✅/⚠️/❌ |
| CLS | X.XX | ≤ 0.1 | ✅/⚠️/❌ |
| TTFB | XXXms | ≤ 600ms | ✅/⚠️/❌ |

### Lighthouse Scores
| Category | Score | Δ vs Last |
|----------|-------|----------|
| Performance | XX | +/-X |
| Accessibility | XX | +/-X |
| Best Practices | XX | +/-X |
| SEO | XX | +/-X |

---

### Findings

#### 🔴 Critical Performance Issues
1. **[Issue]**
   - Metric affected: [LCP/INP/CLS]
   - Current: [measured value]
   - Target: [threshold]
   - Root cause: [specific finding]
   - Fix: [specific action with code/config]
   - Estimated impact: [~Xs improvement in LCP]

#### 🟠 High Impact Optimizations
1. **[Issue]** — [current value] → [expected after fix]

#### 🟡 Medium Impact (next sprint)
1. **[Issue]**

---

### Bundle Analysis
| Route | Current Size | Target | Action |
|-------|-------------|--------|--------|
| / (Home) | XXX KB | < 300 KB | [action] |
| /dashboard | XXX KB | < 200 KB | [action] |

### Verdict
🔴 FAILING Core Web Vitals — SEO and UX impact / 🟡 BORDERLINE — optimize before release / 🟢 PASSING — no critical issues
```
