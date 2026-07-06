# Claude QA Skill

> A production-grade QA agent for Claude — covering the full testing lifecycle from test plans to channel push.

![Version](https://img.shields.io/badge/version-1.2.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Claude](https://img.shields.io/badge/Claude-Skill-orange)
![Stacks](https://img.shields.io/badge/stacks-Next.js%20·%20Flutter%20·%20Supabase%20·%20Convex-purple)

---

## What It Does

Install once. Claude becomes a senior QA engineer that:

- Writes test plans and test cases from a feature brief
- Executes structured test sessions with evidence standards
- Files bug reports with severity classification and reproduction steps
- Reviews code quality across 8 stacks with stack-specific checklists
- Runs OWASP Top 10 security scans on every auth flow and API endpoint
- Audits UX/UI against Nielsen heuristics and Figma specs
- Tests performance against Core Web Vitals (LCP / INP / CLS)
- Writes E2E tests in Playwright or Cypress
- Tests API contracts with Postman/Newman
- Builds risk-based smart test plans with a Requirements Traceability Matrix (RTM)
- Validates data quality and accuracy across 6 dimensions (SQL / Supabase / Convex checks)
- Profiles data structure: schema drift, statistical shape, and 10x volume/scale testing
- Orchestrates full multi-phase QA missions (Super Power mode) with a live mission board
- Delivers a GO/NO-GO release gate verdict
- **Pushes every finding to Discord, Jira, or WhatsApp automatically**

---

## Supported Stacks

| Frontend | Backend | Database | Auth | Runtime |
|----------|---------|----------|------|---------|
| Next.js 14+ / React 18 | Node.js · Express · Fastify · Hono | Supabase (RLS) | Clerk | Bun |
| Flutter (iOS / Android) | tRPC | Convex | NextAuth | Node 20+ |
| shadcn/ui | REST API | Prisma · Drizzle | JWT | — |

---

## Quick Start

### 1 — Install the skill

Download `claude-qa-skill.skill` from [Releases](../../releases) and open it in Claude.ai.  
Click **Save skill** when prompted.

### 2 — Configure

Open `SKILL.md` and fill in the config block at the top:

```
OWNER_NAME     → Your name (appears in all report headers)
OUTPUT_CHANNEL → discord | jira | whatsapp | all
ESCALATION_LEAD → Who receives S1/Critical escalations
```

For channel-specific tokens, see [Channel Setup](#channel-setup) below.

### 3 — Use it

Trigger in plain language — Arabic or English:

```
"test this login flow"
"خطة اختبار لميزة الدفع"
"review this PR for security issues"
"هل نطلق؟"
"write Playwright tests for the checkout flow"
"push to Discord"
```

Claude auto-selects the right mode. No manual routing needed.

---

## Modes Reference

| Mode | What It Does | Trigger Examples |
|------|-------------|-----------------|
| `test-plan` | Writes full test plan + test cases from a brief | "plan this feature", "خطة اختبار" |
| `execute` | Runs structured test session with pass/fail tracking | "test this", "شغّل الاختبار" |
| `bug-report` | Files classified bug report (S1–S4, P0–P3) | "bug", "في خطأ", "مشكلة" |
| `code-review` | Stack-specific code quality review | PR link, code snippet, "راجع الكود" |
| `e2e` | Generates Playwright or Cypress test suite | "write E2E tests", "اكتب اختبارات" |
| `api-contract` | Tests API contracts with Postman/Newman | "test this API", "فحص الـ endpoints" |
| `ux-review` | Nielsen heuristics UX audit | "UX review", "تجربة مستخدم" |
| `ui-audit` | Figma-to-dev gap analysis + visual consistency | "واجهة", Figma link, screenshot |
| `security-scan` | OWASP Top 10 security assessment | "أمان", "ثغرة", "فحص أمني" |
| `performance` | Core Web Vitals + bundle analysis + backend response | "بطيء", "Lighthouse", LCP/INP/CLS |
| `release-gate` | Full GO/NO-GO with 5 gates | "هل نطلق؟", "deploy", "GO/NO-GO" |
| `full-audit` | UX → UI → Security → Performance → Release Gate | "فحص شامل", "audit كامل" |
| `channel-push` | Push any report to Discord/Jira/WhatsApp | "ارسل للقناة", "push to Discord" |

---

## Channel Setup

### Discord (recommended — fastest setup)

1. Discord → Server Settings → Integrations → Webhooks → **New Webhook**
2. Select your `#qa-reports` channel → Copy webhook URL
3. Paste in `references/10_channel-integration.md` under `DISCORD_WEBHOOK_URL`

Every bug report, release decision, and S1 alert pushes automatically as a rich embed with color-coded severity.

### Jira

1. Jira → Profile → Security → **Create API token**
2. Paste `JIRA_DOMAIN`, `JIRA_PROJECT_KEY`, `JIRA_EMAIL`, and `JIRA_API_TOKEN` in `references/10_channel-integration.md`

Bugs create Jira issues with correct priority, labels, and stack name. Jira ticket link auto-embeds in the Discord message.

### WhatsApp

No automation without the Business API. The skill generates formatted text blocks for copy-paste into your group.

---

## Severity System

### Bugs
| Level | SLA |
|-------|-----|
| S1 — Critical | Block release — fix now |
| S2 — High | Fix this sprint |
| S3 — Medium | Next sprint |
| S4 — Low | Backlog |

### Performance
| Metric | Good | Poor |
|--------|------|------|
| LCP | ≤ 2.5s | > 4.0s |
| INP | ≤ 200ms | > 500ms |
| CLS | ≤ 0.1 | > 0.25 |

### Security (OWASP Top 10 — all 10 covered)
Critical → High → Medium → Low with OWASP reference + remediation snippet on every finding.

---

## Reference Files

| File | Coverage |
|------|---------|
| `01_test-plans.md` | Test plan template, test case template, platform matrix, feature checklists |
| `02_execution.md` | Execution report, blocking rules, escalation path, regression protocol |
| `03_bug-reports.md` | Bug report template, severity guide, evidence standards, Linear template |
| `04_code-quality.md` | Next.js, Flutter, Node.js, Supabase, tRPC, Prisma, Drizzle, Clerk, Convex |
| `05_ux-testing.md` | Nielsen heuristics, flow testing, mobile UX, RTL/Arabic, accessibility (WCAG 2.1 AA) |
| `06_release-gate.md` | 5-gate checklist, risk classification, rollback plan, hotfix protocol |
| `07_ui-audit.md` | Figma-to-dev gap, design tokens, component states, cross-browser, dark mode |
| `08_security-testing.md` | OWASP A01–A10, JWT testing, session management, API security, security headers |
| `09_performance-testing.md` | Core Web Vitals, Lighthouse, bundle analysis, backend response times, Flutter FPS |
| `10_channel-integration.md` | Discord webhooks, Jira REST API, WhatsApp templates, emergency push |
| `11_e2e-testing.md` | Playwright + Cypress, Page Object Model, GitHub Actions CI |
| `12_api-contract-testing.md` | Postman/Newman, OpenAPI contract validation, CI pipeline integration |

---

## RTL & Arabic First-Class Support

Arabic-language trigger words work natively. Every checklist includes RTL-specific items:
- Layout mirroring validation
- Arabic font rendering and line height
- `dir="rtl"` and logical CSS properties
- Arabic input on all form fields
- Right-to-left test case documentation

---

## Contributing

PRs welcome. Focus areas for contribution:
- Additional stack checklists (SvelteKit, Nuxt, Expo, etc.)
- More OWASP-specific remediation snippets
- Localization of templates (FR, ES, TR)
- Load testing patterns (k6, Artillery)

---

## License

MIT — free to use, fork, and extend.  
If this skill saves your team time, a star ⭐ is appreciated.

---

Built for teams shipping real products under real pressure.
