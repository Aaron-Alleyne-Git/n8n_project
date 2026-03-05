# 🔁 Iteration Ideas

This project is intentionally minimal. These are the natural next steps — each one adds real-world value and a new skill.

---

## Tier 1 — Easy Wins (1–2 hours each)

### 1. Add Email Alerting
Use the **SMTP node** or **Gmail node** to send email alerts alongside Slack.

- N8N SMTP node docs: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.emailsend/
- Add it in parallel with the Slack node (fan out from the Format node)
- Good for: reaching people who aren't on Slack

### 2. Monitor Multiple Repos
Use the **Split In Batches** or **Loop** pattern to monitor a list of repos.

- Store repo list as a JSON array in a **Set** node
- Loop through with **SplitInBatches**
- Docs: https://docs.n8n.io/flow-logic/looping/

### 3. Add a Manual Trigger
Add a **Manual Trigger** node alongside the Schedule so you can test at any time.

- Docs: https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.manualworkflowtrigger/

---

## Tier 2 — Intermediate (half day each)

### 4. Deduplication with a Database
Right now, the same advisory will alert every 6 hours if it stays unresolved. Fix this by storing seen advisory IDs.

**Options:**
- **Airtable node** (easiest, no infra): https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.airtable/
- **PostgreSQL node** (more realistic): https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.postgres/
- **Redis node** (fast TTL-based dedup): https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.redis/

**Pattern:**
```
After filter → Check DB for advisory ID
  → IF exists: stop
  → IF new: alert + write to DB
```

### 5. Severity-Based Routing
Route critical and high alerts to different destinations.

- Critical → PagerDuty (wake someone up)
- High → Slack
- Add an **IF** node after the filter

**PagerDuty integration:** https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.pagerduty/

### 6. Webhook Trigger (Real-Time)
Replace the polling schedule with a webhook to react instantly when GitHub sends an event.

- GitHub supports webhooks for `security_advisory` events
- N8N webhook docs: https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/
- Requires a public URL — use [ngrok](https://ngrok.com/) for local dev

---

## Tier 3 — Advanced (1–2 days)

### 7. Auto-Create Jira Tickets
For critical advisories, automatically open a Jira issue with pre-filled fields.

- Jira node docs: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.jira/
- Fields to populate: summary, description, priority, labels, assignee

### 8. Sub-Workflow Pattern
Extract the Slack notification logic into a reusable **sub-workflow** that any other workflow can call.

- Docs: https://docs.n8n.io/flow-logic/subworkflows/
- Good pattern for: reusable notification logic across multiple alert workflows

### 9. Error Handling Workflow
Add a dedicated error workflow that alerts you when the main workflow fails.

- N8N error workflow docs: https://docs.n8n.io/flow-logic/error-handling/
- Common causes: GitHub API rate limit, expired token, Slack webhook rotation

### 10. Export to SIEM
Forward structured log data from the Log Execution node to a real SIEM.

- **Datadog**: Use HTTP Request node to POST to Datadog Logs API
- **Splunk HEC**: HTTP Request to Splunk HTTP Event Collector
- **Elastic**: HTTP Request to Elasticsearch ingest endpoint

---

## Portfolio Tip

When you complete a tier, update your README to show what you built. Recruiters look for:
- A working demo (screenshot or short Loom video)
- What problem it solves
- What you'd do differently next time (shows self-awareness)
