# 🛠️ Setup Guide

This guide walks you through getting the N8N DevSecOps Starter running locally from scratch.

---

## Step 1: Install Docker Desktop

N8N runs in Docker, so you need Docker installed first.

- Download: https://www.docker.com/products/docker-desktop/
- After install, verify it works:

```bash
docker --version
docker compose version
```

---

## Step 2: Create a GitHub Personal Access Token

The workflow reads security advisories from GitHub. You need a token to authenticate.

1. Go to: https://github.com/settings/tokens
2. Click **Generate new token (classic)**
3. Name it: `n8n-devsecops-starter`
4. Set expiry: 90 days (rotate regularly)
5. Check **only** this scope:
   - ✅ `public_repo`
6. Click **Generate token** and copy it immediately

> ⚠️ **Security note:** Minimum required scope only. Never give a token more permissions than it needs.

---

## Step 3: Create a Slack Webhook

1. Go to: https://api.slack.com/apps
2. Click **Create New App → From Scratch**
3. Name it: `Security Alerts`
4. Select your workspace
5. Under **Features**, click **Incoming Webhooks**
6. Toggle it **On**
7. Click **Add New Webhook to Workspace**
8. Choose a channel (e.g., `#security-alerts`)
9. Copy the Webhook URL

---

## Step 4: Configure Environment Variables

```bash
# Clone the repo (or just use your local copy)
cd n8n-devsecops-starter

# Copy the template
cp .env.example .env

# Edit .env
nano .env  # or code .env / vim .env
```

Fill in all values in `.env`. Generate the encryption key like this:

```bash
openssl rand -hex 32
```

---

## Step 5: Start N8N

```bash
docker compose up -d
```

Check it's running:

```bash
docker compose ps
docker compose logs n8n
```

Open in browser: http://localhost:5678

Log in with the credentials from your `.env` file.

---

## Step 6: Import the Workflow

1. In N8N, click **Workflows** in the left sidebar
2. Click **Add Workflow → Import from File**
3. Select `workflows/security-advisory-alert.json`
4. The workflow will open in the editor
5. Click **Save**

---

## Step 7: Test It

1. In the workflow editor, click **Execute Workflow** (▶️ button)
2. Watch the nodes light up as they execute
3. Check your Slack channel for an alert message

> If no advisories come through, it may mean the monitored repo has no HIGH/CRITICAL advisories. Try changing `GITHUB_REPO_NAME` to a large public repo like `kubernetes/kubernetes`.

---

## Step 8: Activate the Schedule

Once testing passes:

1. Toggle the **Active** switch in the top right of the workflow editor
2. The workflow will now run every 6 hours automatically

---

## Stopping N8N

```bash
docker compose down
```

Data is preserved in the Docker volume. Next time you run `docker compose up -d`, everything will be as you left it.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| N8N won't start | Check `docker compose logs n8n` for errors |
| GitHub API returns 401 | Token is invalid or expired — regenerate it |
| GitHub API returns 403 | Token doesn't have `public_repo` scope |
| Slack alert not received | Check webhook URL is correct in `.env` |
| Workflow shows no items | Repo may have no HIGH/CRITICAL advisories — try a different repo |
