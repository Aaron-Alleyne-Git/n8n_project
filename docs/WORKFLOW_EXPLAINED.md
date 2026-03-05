# 🔍 Workflow Explained: Node-by-Node

This document explains every node in `security-advisory-alert.json` so you understand what's happening and why.

---

## Visual Overview

```
[1. Schedule Trigger]
        ↓
[2. HTTP Request → GitHub Advisory API]
        ↓
[3. Filter: HIGH or CRITICAL only]
        ↓
[4. Code: Format Slack Message]
        ↓
[5. HTTP Request → Slack Webhook]
        ↓
[6. Code: Log Execution]
```

---

## Node 1: Schedule Trigger

**Type:** `n8n-nodes-base.scheduleTrigger`

**What it does:** Starts the workflow automatically every 6 hours.

**Why 6 hours?** A balance between timely alerts and API rate limit conservation. GitHub's REST API allows 5,000 requests/hour for authenticated users — polling every 6 hours is very conservative.

**Key concept:** N8N workflows need a *trigger* node to start. Options include:
- Schedule (time-based, like this one)
- Webhook (HTTP event-based)
- Manual (you click a button)
- App triggers (new GitHub issue, new email, etc.)

**Reference:** https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.scheduletrigger/

---

## Node 2: HTTP Request → GitHub Advisory API

**Type:** `n8n-nodes-base.httpRequest`

**What it does:** Makes a GET request to the GitHub Security Advisories API for the configured repository.

**API endpoint:**
```
GET https://api.github.com/repos/{owner}/{repo}/security-advisories
```

**Security design choices:**
- Auth token is read from `$env.GITHUB_TOKEN` — not hardcoded in the node
- Headers include the required `X-GitHub-Api-Version` for stable API behavior
- Timeout is set (10 seconds) to prevent hanging executions

**What the response looks like:**
```json
[
  {
    "ghsa_id": "GHSA-xxxx-xxxx-xxxx",
    "cve_id": "CVE-2024-12345",
    "summary": "Critical buffer overflow in libfoo",
    "severity": "critical",
    "published_at": "2024-01-15T10:00:00Z",
    "html_url": "https://github.com/advisories/GHSA-xxxx"
  }
]
```

**Reference:** https://docs.github.com/en/rest/security-advisories/repository-advisories

---

## Node 3: Filter — HIGH or CRITICAL Only

**Type:** `n8n-nodes-base.filter`

**What it does:** Only passes advisory items where `severity` equals `"high"` OR `"critical"`. All other items are discarded.

**Why filter here?** This is *alert fatigue prevention*. If every low-severity advisory triggered a Slack message, teams would start ignoring alerts. High/critical advisories require immediate action — medium and low can be reviewed in weekly reports.

**How to adjust:** Open the Filter node and change the condition values. You can add `"medium"` if your team wants wider coverage.

**Key N8N concept:** The Filter node outputs items that *pass* the condition. Items that don't match simply stop flowing through the workflow.

**Reference:** https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.filter/

---

## Node 4: Code — Format Slack Message

**Type:** `n8n-nodes-base.code`

**What it does:** Transforms the raw GitHub advisory JSON into a Slack Block Kit message format.

**Why a Code node?** The GitHub API response format doesn't directly match what Slack expects. This node is a *transformation* step — a common pattern in any integration pipeline.

**Slack Block Kit** is Slack's structured message format. It produces rich, interactive messages with sections, buttons, and headers instead of plain text.

**The transformation:**
```
GitHub Advisory Object → Slack Block Kit Payload
{ severity, cve_id, summary, html_url } → { blocks: [...] }
```

**Reference:** https://api.slack.com/block-kit

---

## Node 5: HTTP Request → Slack Webhook

**Type:** `n8n-nodes-base.httpRequest`

**What it does:** POSTs the formatted Block Kit payload to the Slack Incoming Webhook URL.

**Security design choices:**
- Webhook URL is read from `$env.SLACK_WEBHOOK_URL` — not hardcoded
- Timeout is set (5 seconds) — Slack is fast; if it takes longer, something is wrong
- No credentials are stored inside N8N for this node (webhook URL IS the credential)

**Reference:** https://api.slack.com/messaging/webhooks

---

## Node 6: Code — Log Execution

**Type:** `n8n-nodes-base.code`

**What it does:** Writes a structured JSON log entry to N8N's execution log after each alert is sent.

**Example log output:**
```json
{
  "event": "advisory_alert_sent",
  "timestamp": "2024-01-15T10:00:00.000Z",
  "severity": "critical",
  "cve_id": "CVE-2024-12345",
  "status": "success"
}
```

**Why log this?** Audit trails. In a DevSecOps context, you need to prove that alerts were generated and sent. This data can be:
- Forwarded to a SIEM (Splunk, Datadog, etc.)
- Written to CloudWatch Logs
- Stored in a database for compliance reporting

**Reference:** https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.code/

---

## Key N8N Concepts This Workflow Teaches

| Concept | Where Used |
|---|---|
| **Trigger nodes** | Schedule Trigger (Node 1) |
| **HTTP Request node** | GitHub API call, Slack webhook (Nodes 2, 5) |
| **Data filtering** | Filter node (Node 3) |
| **Data transformation** | Code nodes (Nodes 4, 6) |
| **Environment variables** | `$env.VARIABLE_NAME` pattern |
| **Node chaining** | Data flows automatically from node to node |
| **Execution logs** | N8N stores each run in Executions panel |
