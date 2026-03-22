# Retail Tech Ops Daily Dashboard

Live standup dashboard for Arc'teryx Retail Technology operations.  
Auto-pulls from **Jira (project: RTS)** and **PagerDuty** every 15 min via GitHub Actions. RAG status and Order Messages are entered manually in the browser and persist via localStorage.

---

## Repo structure

```
/
├── index.html                        ← the dashboard
├── data.json                         ← auto-written by GitHub Actions (do not edit)
├── .github/
│   └── workflows/
│       └── fetch-ops-data.yml
└── README.md
```

---

## Setup

### 1. GitHub Secrets
Settings → Secrets and variables → Actions:

| Secret | Value |
|--------|-------|
| `JIRA_BASE_URL` | `https://arcteryx.atlassian.net` |
| `JIRA_EMAIL` | your Jira login email |
| `JIRA_API_TOKEN` | Jira personal API token (id.atlassian.com) |
| `PD_API_TOKEN` | PagerDuty personal REST API token |
| `PD_SCHEDULE_ID` | PagerDuty schedule UUID (from schedule URL) |

Note: `JIRA_JSM_PROJECT` and `JIRA_SW_PROJECT` are hardcoded to `RTS` in the workflow. Update there if the project key changes.

### 2. GitHub Pages
Settings → Pages → Source: branch `main`, folder `/ (root)`.

Dashboard URL: `https://<your-org>.github.io/<repo-name>/`

### 3. First run
Actions → Fetch Ops Data → Run workflow to generate `data.json` immediately.

---

## Jira config (project: RTS)

| Metric | JQL |
|--------|-----|
| Currently Open | `project = RTS AND status != Done` |
| P1 (Blocker/ASAP) | `project = RTS AND priority in (Blocker, ASAP) AND status != Done` |
| P2 (High) | `project = RTS AND priority = High AND status != Done` |
| Open Problems | `project = RTS AND issuetype = Problem AND status != Done` |
| On-Call Tickets | `project = RTS AND status != Done AND labels = on-call` |

**Verify before first run:** Confirm `on-call` is the actual label name used in RTS. If your on-call label differs, update the workflow.

---

## Manual inputs (localStorage)

These are entered directly in the browser and persist per-device:

- **RAG status** — Healthy / Degraded / Outage + notes + updated-by for each system (Digital Displays, Store Music, Network, Traffic System, Adyen Devices, VoIP, POS)
- **Order Messages** — Total Open, Opened 24h, Closed 24h (click any card to update)

For a single dedicated standup laptop this works perfectly. If you later need shared state across browsers, the next step is a Cloudflare Worker + KV store (~1h of work).

---

## Local development
Open `index.html` directly in browser. Runs in mock-data mode (`USE_MOCK = true`) by default — no API calls. Set `USE_MOCK = false` once `data.json` is live.
