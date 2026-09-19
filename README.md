# Lead Processing Automation Workflow

An end-to-end **n8n automation workflow** that captures, validates, enriches, scores, and routes inbound leads — from form submission all the way through to follow-up emails, CRM logging, and manager approvals.

---

## Overview

```
Form Submission → Email Validation → Hunter Enrichment → CRM Lookup → Lead Scoring → Smart Routing
```

This workflow automates the entire lead lifecycle:

| Stage | What happens |
|-------|-------------|
| **Capture** | Contact form triggers the workflow |
| **Validate** | Basic email format check + Hunter.io deliverability verification |
| **Enrich** | Fetch company, role & social data via Hunter API |
| **Unify** | Merge enriched data with existing Airtable CRM records |
| **Score** | Multi-factor scoring: email quality, service region, industry match |
| **Route** | High / Medium / Low value leads get different follow-up actions |
| **Notify** | Slack alerts, calendar events, welcome emails, follow-up sequences |
| **Log** | Every lead and decision is logged to Google Sheets |

---

## Workflow Phases

### Phase 1 — Capture, Validate & Enrich

![Phase 1](phase%201%20of%20workflow.png)

1. **On form submission** — n8n Form Trigger collects: Name, Email, Website, Job Title, Interested Area
2. **Basic email verification** — Regex-based format validation
3. **verify Email With Hunter** — Hunter.io API call for deliverability score
4. **If (score > 20 & status = valid)** — Pass/fail branching
5. **Enrich lead Information** — Hunter Combined Find API for company + person data
6. **unified-Profile** — Merge form data with Hunter enrichment
7. **Search records** — Look up existing contact in Airtable CRM
8. **unified-enriched-profile-with-CRM-data** — Final enriched profile merge
9. **Switch (New-Customer / General / VIP)** — Segment by CRM history & revenue

### Phase 2 — Lead Scoring & Qualification

![Phase 2](phase%202%20of%20workflow.png)

| Step | Node | Logic |
|------|------|-------|
| Tag customer | `Tag-Customer-New` / `Tag-General` / `Tag-VIP` | Assign segment label |
| Initialize score | `Lead Score intialize` | Start at 0 |
| Email quality | `Email Score` | +1 if Hunter score > 60 |
| Region check | `Check Service Region` | +1 if country in US, DE, IN, UK |
| Industry check | `Check Industry` | +1 if industry = Software |
| Classify | `customer Status` | Score ≤ 1 → Low / 2 → Medium / 3 → High |
| Route | `Route lead value` | Branch by lead type |

### Phase 3 — Action, Approval & Follow-up

![Phase 3](phase%203%20of%20workflow.png)

**High Value Lead:**
- `Copy Template` → Google Drive template copy
- `Replace dynamic Data` → Inject company name & contact name
- `Download as PDF` → Export final document
- `Upload file` → Save to project folder
- `Delete a file` → Clean up temp doc
- `Alert account executive` → Slack notification
- `Create an event` → Google Calendar meeting (5 days out)
- `Welcome Email` → Gmail welcome message
- `Follow up` → Gmail follow-up sequence
- `Log Everthing` → Google Sheets audit trail

**Medium Value Lead:**
- `Send message and wait for response` — Manager approval email
- `Manager Approved ?` — If yes → proceed; if no → `Log Everthing1` (disapproved)

**Low Value Lead:**
- `further nurture lead` — Log for nurture sequence

---

## Tech Stack

| Service | Purpose |
|---------|---------|
| [n8n](https://n8n.io) | Workflow automation engine |
| [Hunter.io](https://hunter.io) | Email verification & lead enrichment |
| [Airtable](https://airtable.com) | CRM / contact database |
| [Google Sheets](https://sheets.google.com) | Logging & audit trail |
| [Google Drive](https://drive.google.com) | Template management & PDF storage |
| [Google Docs](https://docs.google.com) | Dynamic document generation |
| [Google Calendar](https://calendar.google.com) | Meeting scheduling |
| [Gmail](https://mail.google.com) | Email notifications & sequences |
| [Slack](https://slack.com) | Real-time team alerts |

---

## Project Structure

```
n8n-lead-automation/
├── README.md
├── .gitignore
├── Lead Processing Workflow.json          # n8n workflow export (import directly)
├── phase 1 of workflow.png                # Phase 1 screenshot
├── phase 2 of workflow.png                # Phase 2 screenshot
├── phase 3 of workflow.png                # Phase 3 screenshot
├── PS/                                    # Detailed workflow screenshots
│   ├── PS_01.png
│   ├── PS_02.png
│   ├── PS_03.png
│   └── PS_04.png
└── lead-generation-workflow/
    ├── templates/
    │   └── qualified_lead_brief_template.md
    └── mock-data/
        ├── airtable_crm_mock_data.csv
        └── form_submission_mock_data.json
```

---

## Getting Started

### Prerequisites

- n8n instance (self-hosted or cloud)
- Hunter.io API key
- Airtable Personal Access Token
- Google account with Sheets, Drive, Docs, Calendar, and Gmail enabled
- Slack workspace with bot token

### Setup

1. **Import the workflow** — Open n8n → Workflows → Import from File → select `Lead Processing Workflow.json`

2. **Configure credentials** in n8n:
   - Hunter API
   - Airtable (OAuth2)
   - Google Sheets (OAuth2)
   - Google Drive (OAuth2)
   - Google Docs (OAuth2)
   - Google Calendar (OAuth2)
   - Gmail (OAuth2)
   - Slack (API)

3. **Update the Google Sheets document ID** — Replace the placeholder spreadsheet ID with your own

4. **Update the Airtable base/table IDs** — Point to your CRM base

5. **Replace the Hunter API key** — Add your own key in the HTTP Request node

6. **Update Slack channel** — Set the `channelId` in the Alert node

7. **Activate the workflow** — Toggle the workflow to Active

---

## Lead Scoring Logic

| Factor | Points | Condition |
|--------|--------|-----------|
| Email quality | +1 | Hunter score > 60 |
| Service region | +1 | Country is US, DE, IN, or UK |
| Industry match | +1 | Industry = Software |
| **Total** | **0–3** | |

| Score | Lead Type | Action |
|-------|-----------|--------|
| 0–1 | Low Value | Log for nurture |
| 2 | Medium Value | Manager approval required |
| 3 | High Value | Full automated follow-up |

---

## Mock Data

Sample data is provided in `lead-generation-workflow/mock-data/` for testing:

- **`airtable_crm_mock_data.csv`** — 108 contacts with name, email, age, gender, feedback, rating, and revenue
- **`form_submission_mock_data.json`** — Sample form payloads to test the workflow trigger

---

## License

MIT
