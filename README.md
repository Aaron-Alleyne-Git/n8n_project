# 🔄 N8N DevSecOps Starter Project

> A beginner-friendly N8N workflow automation project designed with security-first principles.  
> Built for learning, iterating, and showcasing in a DevSecOps role.

---

## 📌 What Is This?

This project uses **[N8N](https://n8n.io/)** — an open-source workflow automation tool — to automate a common DevSecOps task:

**Monitoring a public GitHub repository for new security advisories and alerting a Slack channel.**

It's simple enough to understand quickly, but realistic enough to put on your portfolio.

---

## 🗺️ Project Structure

```
n8n-devsecops-starter/
├── README.md                  ← You are here
├── docker-compose.yml         ← Run N8N locally (secure config)
├── .env.example               ← Environment variable template (never commit .env!)
├── .gitignore
├── workflows/
│   └── security-advisory-alert.json   ← The N8N workflow (importable)
├── docs/
│   ├── SETUP.md               ← Step-by-step setup guide
│   ├── WORKFLOW_EXPLAINED.md  ← How the workflow works
│   └── ITERATION_IDEAS.md     ← Ways to extend this project
└── .github/
    └── workflows/
        └── lint-workflow.yml  ← CI check: validate N8N JSON on push
```

---

## 🔐 Security First (Shift Left)

Security is baked in from the start — not bolted on after.

| Practice | How It's Applied Here |
|---|---|
| **No secrets in code** | All credentials use `.env` variables |
| **Least privilege** | GitHub token scoped to `read:packages` only |
| **Secret scanning** | `.gitignore` prevents `.env` from being committed |
| **Input validation** | Workflow uses N8N's built-in data filtering before sending alerts |
| **Audit trail** | Workflow logs execution timestamps and payload hashes |

---

## 🚀 Quick Start

### Prerequisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed
- A free [N8N Cloud account](https://n8n.io/cloud/) OR Docker (local)
- A [GitHub Personal Access Token](https://github.com/settings/tokens) (scoped to `public_repo`)
- A [Slack Webhook URL](https://api.slack.com/messaging/webhooks)

### 1. Clone and configure

```bash
git clone https://github.com/YOUR_USERNAME/n8n-devsecops-starter.git
cd n8n-devsecops-starter

# Copy the environment template
cp .env.example .env

# Edit .env with your real values (never commit this file!)
nano .env
```

### 2. Start N8N locally

```bash
docker compose up -d
```

N8N will be available at: **http://localhost:5678**

### 3. Import the workflow

1. Open N8N in your browser
2. Go to **Workflows → Import from File**
3. Select `workflows/security-advisory-alert.json`
4. Click **Activate**

### 4. Test it

Trigger the workflow manually in the N8N UI — you should see a Slack message within seconds.

---

## 🔗 Key Resources

| Resource | Link |
|---|---|
| N8N Documentation | https://docs.n8n.io |
| N8N Node Library | https://n8n.io/integrations |
| GitHub Advisory API | https://docs.github.com/en/rest/security-advisories |
| Slack Incoming Webhooks | https://api.slack.com/messaging/webhooks |
| Docker Compose Docs | https://docs.docker.com/compose/ |
| OWASP Secrets Management | https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html |

---

## 🧩 What the Workflow Does

```
[Schedule: Every 6 hours]
        ↓
[HTTP Request → GitHub Advisory API]
        ↓
[Filter: Only HIGH or CRITICAL severity]
        ↓
[Format Message]
        ↓
[POST → Slack Webhook]
        ↓
[Log Execution Result]
```

See [`docs/WORKFLOW_EXPLAINED.md`](docs/WORKFLOW_EXPLAINED.md) for a full node-by-node breakdown.

---

## 🔁 Iteration Roadmap

This project is intentionally minimal. Here's what to build next:

- [ ] Add email alerting (SMTP node) alongside Slack
- [ ] Store advisory history in Airtable or PostgreSQL (avoid duplicate alerts)
- [ ] Add severity-based routing (Critical → PagerDuty, High → Slack)
- [ ] Build a webhook trigger to react in real-time instead of polling
- [ ] Add a sub-workflow to auto-create Jira tickets for critical advisories

See [`docs/ITERATION_IDEAS.md`](docs/ITERATION_IDEAS.md) for implementation notes.

---

## 🧠 Why This Project for DevSecOps?

| Skill Demonstrated | How |
|---|---|
| **Workflow automation** | Built an end-to-end N8N pipeline |
| **API integration** | Consumed GitHub REST API with auth |
| **Security hygiene** | Environment variables, least privilege, secret scanning |
| **Alerting pipelines** | Slack webhook integration |
| **Documentation** | README, setup guide, architecture explanation |
| **CI/CD basics** | GitHub Actions workflow to validate JSON |

---

## 📄 License

MIT — use freely, learn openly.