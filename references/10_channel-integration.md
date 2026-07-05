# 10 — Channel Integration: Discord · Jira · WhatsApp

Push every finding the moment it's documented. No report stays local.

---

## Channel Configuration

Set your values here once. The skill reads from SKILL.md config + this file.

```
# Discord
DISCORD_WEBHOOK_URL = https://discord.com/api/webhooks/YOUR_ID/YOUR_TOKEN
DISCORD_CHANNEL_NAME = #qa-reports     # for reference only

# Jira
JIRA_DOMAIN         = yourteam.atlassian.net
JIRA_PROJECT_KEY    = QA
JIRA_EMAIL          = ghassan@yourteam.com
JIRA_API_TOKEN      = YOUR_JIRA_API_TOKEN

# WhatsApp
WHATSAPP_GROUP_NAME = QA Team          # for reference only — manual push
```

> **Security note:** Never commit tokens to git. Store in `.env.local` or a secrets manager.

---

## When to Push

| Event | Push Trigger | Channel |
|-------|-------------|---------|
| Bug found (S1/S2) | Immediately after documenting | Discord + Jira |
| Bug found (S3/S4) | After test session ends | Discord |
| Security Critical/High | Immediately — emergency push | Discord + Jira |
| Execution Report complete | End of test session | Discord |
| Release Gate decision | GO or NO-GO finalized | Discord + Jira |
| Full Audit complete | After executive summary | Discord + Jira |
| Code Review complete | After verdict | Discord |

---

## Discord Integration

### Setup (one time)
1. Discord → Server Settings → Integrations → Webhooks → New Webhook
2. Select channel (`#qa-reports` or `#bugs`)
3. Copy Webhook URL → paste in config above

### Message Format by Report Type

#### Bug Report Push

Generate this payload after every bug report:

```json
{
  "username": "QA Bot — Ghassan Ahmed",
  "avatar_url": "https://cdn-icons-png.flaticon.com/512/1828/1828843.png",
  "embeds": [{
    "title": "🐛 BUG-[XXX] — [Bug Title]",
    "description": "[One-line description of the bug]",
    "color": "[COLOR_CODE]",
    "fields": [
      { "name": "Severity", "value": "S[X] — [Level]", "inline": true },
      { "name": "Priority", "value": "P[X]", "inline": true },
      { "name": "Status", "value": "🔴 Open", "inline": true },
      { "name": "Platform", "value": "[Web / iOS / Android / API]", "inline": true },
      { "name": "Stack", "value": "[Detected stack]", "inline": true },
      { "name": "Build", "value": "[v1.x.x]", "inline": true },
      { "name": "Steps to Reproduce", "value": "1. [Step]\n2. [Step]\n3. [Step]", "inline": false },
      { "name": "Expected", "value": "[Expected result]", "inline": true },
      { "name": "Actual", "value": "[Actual result]", "inline": true }
    ],
    "footer": { "text": "QA Report by Ghassan Ahmed • [YYYY-MM-DD HH:MM]" },
    "timestamp": "[ISO8601_TIMESTAMP]"
  }]
}
```

**Color codes by severity:**
| Severity | Discord Color (decimal) | Hex |
|----------|------------------------|-----|
| S1 Critical | `16711680` | `#FF0000` |
| S2 High | `16744272` | `#FF7110` |
| S3 Medium | `16776960` | `#FFFF00` |
| S4 Low | `5763719` | `#57F287` |
| Security Critical | `10038562` | `#990042` |

**Push command (bash):**
```bash
curl -H "Content-Type: application/json" \
  -X POST \
  -d '[PAYLOAD_JSON]' \
  "[DISCORD_WEBHOOK_URL]"
```

---

#### Release Gate Push

```json
{
  "username": "QA Bot — Ghassan Ahmed",
  "embeds": [{
    "title": "[🟢 GO / 🔴 NO-GO] — Release [Version]",
    "description": "[One paragraph decision summary]",
    "color": "[5763719 for GO | 16711680 for NO-GO]",
    "fields": [
      { "name": "Quality", "value": "[✅/❌] Zero S1/S2 open", "inline": true },
      { "name": "Security", "value": "[✅/❌] Zero Critical/High", "inline": true },
      { "name": "Performance", "value": "[✅/❌] Lighthouse ≥ 80", "inline": true },
      { "name": "Open Items", "value": "[X] S3 bugs tracked\n[X] S4 bugs in backlog", "inline": false },
      { "name": "Signed by", "value": "Ghassan Ahmed — QA Lead", "inline": true }
    ],
    "footer": { "text": "[Project Name] Release Gate • [YYYY-MM-DD]" }
  }]
}
```

---

#### Execution Report Summary Push

```json
{
  "username": "QA Bot — Ghassan Ahmed",
  "embeds": [{
    "title": "📊 Test Execution Report — [Feature / Sprint]",
    "description": "**Pass Rate:** [XX]% | **Build:** [v1.x.x] | **Environment:** [Staging]",
    "color": "[color based on pass rate: ≥90% green, 70-89% yellow, <70% red]",
    "fields": [
      { "name": "✅ Passed", "value": "[XX]", "inline": true },
      { "name": "❌ Failed", "value": "[XX]", "inline": true },
      { "name": "⚠️ Blocked", "value": "[XX]", "inline": true },
      { "name": "🔴 Critical/P0", "value": "[list or 'None']", "inline": false },
      { "name": "🟠 High/P1", "value": "[list or 'None']", "inline": false },
      { "name": "Overall Status", "value": "[🔴 FAIL / 🟡 CONDITIONAL / 🟢 PASS]", "inline": false }
    ],
    "footer": { "text": "QA Report by Ghassan Ahmed • [YYYY-MM-DD]" }
  }]
}
```

---

#### Emergency Push (S1 / Security Critical)

Send immediately — don't wait for full report:

```json
{
  "content": "@here 🚨 **CRITICAL ISSUE FOUND — STOP RELEASE**",
  "username": "QA Bot — Ghassan Ahmed",
  "embeds": [{
    "title": "🚨 [S1 CRITICAL / SEC-CRITICAL] — [Issue Title]",
    "description": "[What broke / What vulnerability found]",
    "color": 16711680,
    "fields": [
      { "name": "Impact", "value": "[Who is affected and how]", "inline": false },
      { "name": "Reproduction", "value": "[Fastest way to reproduce]", "inline": false },
      { "name": "Release Status", "value": "🔴 BLOCKED — Do NOT deploy", "inline": false },
      { "name": "Escalated To", "value": "[ESCALATION_LEAD]", "inline": true }
    ],
    "footer": { "text": "Emergency Alert by Ghassan Ahmed • [TIMESTAMP]" }
  }]
}
```

---

## Jira Integration

### Setup (one time)
1. Jira → Profile → Security → Create API token
2. Paste in config above
3. Confirm your project key (e.g. `QA`, `BUG`, `DEV`)

### Create Bug Issue

```bash
curl -X POST \
  -H "Authorization: Basic $(echo -n 'EMAIL:API_TOKEN' | base64)" \
  -H "Content-Type: application/json" \
  --data '{
    "fields": {
      "project": { "key": "[JIRA_PROJECT_KEY]" },
      "summary": "BUG-[XXX] — [Bug Title]",
      "description": {
        "type": "doc",
        "version": 1,
        "content": [{
          "type": "paragraph",
          "content": [{
            "type": "text",
            "text": "[Bug description]\n\nSteps:\n1. [Step]\n2. [Step]\n\nExpected: [result]\nActual: [result]\n\nEnvironment: [platform, browser, version]"
          }]
        }]
      },
      "issuetype": { "name": "Bug" },
      "priority": { "name": "[Highest / High / Medium / Low]" },
      "labels": ["qa", "[stack-name]", "[S1/S2/S3/S4]"],
      "assignee": { "accountId": "[DEV_ACCOUNT_ID]" }
    }
  }' \
  "https://[JIRA_DOMAIN]/rest/api/3/issue"
```

### Priority Mapping

| QA Severity | Jira Priority |
|-------------|--------------|
| S1 Critical | Highest |
| S2 High | High |
| S3 Medium | Medium |
| S4 Low | Low |

### Create Security Finding Issue

```bash
# Same as above but with these fields changed:
"issuetype": { "name": "Bug" },
"labels": ["security", "owasp", "[Critical/High/Medium/Low]"],
"summary": "SEC-[XXX] — [Vulnerability Title] — [OWASP A0X]",
"priority": { "name": "Highest" }  # Always Highest for Critical/High security
```

### Link Jira Issue in Discord

After creating a Jira issue, the API returns the issue key (e.g. `QA-47`).  
Add it to the Discord embed:

```json
{ "name": "Jira Ticket", "value": "[QA-47](https://[JIRA_DOMAIN]/browse/QA-47)", "inline": true }
```

---

## WhatsApp Integration

WhatsApp does not support incoming webhooks or bot automation without the Business API (paid, requires Meta approval).

**For WhatsApp groups:** Use the formatted template below — copy and paste.

### Bug Alert Template (WhatsApp)

```
🐛 *BUG REPORT — [Project Name]*
━━━━━━━━━━━━━━━━━━━
*ID:* BUG-[XXX]
*Title:* [Bug Title]
*Severity:* 🔴 S1 / 🟠 S2 / 🟡 S3 / 🟢 S4
*Priority:* P[X]
*Platform:* [Web / iOS / Android]
*Build:* v[X.X.X]
━━━━━━━━━━━━━━━━━━━
*Steps:*
1. [Step]
2. [Step]
3. [Observed result]

*Expected:* [What should happen]
*Actual:* [What actually happened]
━━━━━━━━━━━━━━━━━━━
*Reporter:* Ghassan Ahmed
*Date:* [YYYY-MM-DD HH:MM]
```

### Release Gate Template (WhatsApp)

```
✅ *RELEASE GATE — [Version]*
━━━━━━━━━━━━━━━━━━━
*Decision:* 🟢 GO / 🔴 NO-GO
*Quality:* [✅/❌] Zero S1/S2
*Security:* [✅/❌] Zero Critical/High
*Performance:* [✅/❌] Lighthouse ≥ 80
*Open Items:* [X] tracked, non-blocking
━━━━━━━━━━━━━━━━━━━
*Signed:* Ghassan Ahmed — QA
*Date:* [YYYY-MM-DD]
```

---

## Workflow: Test → Report → Push

```
1. Skill runs mode (bug-report / security-scan / release-gate / etc.)
   ↓
2. Generates structured Markdown report
   ↓
3. Reads OUTPUT_CHANNEL from config
   ↓
4. If OUTPUT_CHANNEL = discord  → Generate Discord embed JSON + curl command
   If OUTPUT_CHANNEL = jira     → Generate Jira issue JSON + curl command
   If OUTPUT_CHANNEL = whatsapp → Generate formatted WA text block
   If OUTPUT_CHANNEL = all      → Generate all three
   ↓
5. For S1 / Security Critical:
   → Generate emergency push payload FIRST, before full report
   → Mention @here in Discord
   → Set Jira priority = Highest
   ↓
6. After push: report complete ✅
```

---

## Quick Reference: Severity → Channel Action

| Finding Type | Discord | Jira | WhatsApp |
|-------------|---------|------|---------|
| S1 Bug | 🔴 @here embed | ✅ Highest issue | ✅ Immediate paste |
| S2 Bug | 🟠 embed | ✅ High issue | Optional |
| S3/S4 Bug | 🟡/🟢 embed (batch) | ✅ Medium/Low | No |
| Security Critical | 🔴 @here embed | ✅ Highest + `security` label | ✅ Immediate |
| Security High | 🟠 embed | ✅ High + `security` label | No |
| Execution Report | 📊 summary embed | No | Optional |
| GO decision | 🟢 embed | ✅ Close sprint task | Optional |
| NO-GO decision | 🔴 @here embed | ✅ Block release task | ✅ Immediate |
