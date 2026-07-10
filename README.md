# Unified Lead Capture → CRM → Nurture Pipeline

An n8n workflow that captures leads from multiple channels, scores and routes them, syncs them to a CRM/database, triggers nurture emails, and sends a weekly summary report.

## What it does

**1. Multi-channel intake**
Three webhooks receive leads from different sources:
- `POST /website-lead` — website contact form
- `POST /linkedin-lead` — LinkedIn Lead Gen forms
- `POST /whatsapp-lead` — WhatsApp Business API messages

**2. Normalization**
Each source has its own **Code node** that maps its raw payload into a common shape:
`name, email, phone, company, company_size, message, source, engagement_signal, received_at`.

**3. Lead scoring** (`Score Lead`)
Computes a 0–100 score from company size, lead source, and engagement signals (message length, presence of email/phone), then buckets leads into `hot` (≥70), `warm` (≥40), or `cold`.

**4. CRM / database sync**
Every scored lead is written in parallel to:
- **HubSpot** (contact created/updated with lead source as a custom property)
- **Airtable** (record with name, contact info, score, temperature, company size)

**5. Routing & instant response**
- `Is Hot Lead?` checks temperature — if `hot`, posts an alert to the **#hot-leads** Slack channel with full lead details.
- Every lead gets a **Send Welcome Email** (Gmail) and a **Send Welcome WhatsApp** message (via Facebook Graph API) immediately.

**6. Nurture sequence**
`Send Welcome Email` → wait 3 days → `Send Follow-up Email` → wait 4 days → `Send Offer Email1` (special-offer email referencing the lead's company).

**7. Weekly reporting**
A **Schedule Trigger** (every Monday, 9am) pulls all HubSpot contacts, tallies them by source and temperature in a Code node, and sends the summary via **Gmail** (to `sales.manager@yourcompany.com`) and **Slack** (`#sales-pipeline`).

## Flow diagram (simplified)

```
Website Webhook   ─┐
LinkedIn Webhook  ─┼─> Normalize (per-source) ─> Score Lead ─┬─> Add to HubSpot
WhatsApp Webhook  ─┘                                          ├─> Add to Airtable
                                                               ├─> Is Hot Lead? ─> Slack #hot-leads
                                                               ├─> Send Welcome Email ─> Wait 3d ─> Follow-up Email ─> Wait 4d ─> Offer Email
                                                               └─> Send Welcome WhatsApp

Weekly Schedule ─> Get Leads (HubSpot) ─> Build Summary ─> Send Report Email
                                                        └─> Send Report Slack (#sales-pipeline)
```

## Required credentials

| Service | Credential type | Used for |
|---|---|---|
| Gmail | OAuth2 | Welcome, follow-up, offer, and weekly report emails |
| HubSpot | OAuth2 | Contact creation + weekly lead pull |
| Airtable | OAuth2 | Lead record storage |
| Slack | OAuth2 | Hot-lead alerts + weekly report |
| WhatsApp/Facebook Graph API | Header Auth | Sending WhatsApp welcome messages |

## Setup notes

- Replace `YOUR_PHONE_NUMBER_ID` in the **Send Welcome WhatsApp** node's URL with your actual WhatsApp Business phone number ID.
- Update the Airtable `base`/`table` IDs (`appA3WFW0sgtBO7nf` / `tblIAcIgdkNwTTlsT`) to point at your own base.
- Update the hard-coded weekly-report recipient (`sales.manager@yourcompany.com`) and Slack channel names (`#hot-leads`, `#sales-pipeline`) to match your workspace.
- The workflow is currently **inactive** (`"active": false`) — activate it in n8n once credentials and IDs above are set.

## ⚠️ Orphan nodes

The workflow contains a **"Wait 4 Days" → "Send Offer Email"** pair that is not connected to any trigger — nothing feeds into "Wait 4 Days", so this branch will never execute. It appears to be a leftover duplicate of the connected **"Wait 4 Days1" → "Send Offer Email1"** pair in the nurture sequence. You can safely delete the orphaned pair, or rewire it if it was meant for a different purpose.

## Lead scoring formula

```
score = company_size_points + source_points
      + (15 if message length > 20 chars)
      + (10 if email present)
      + (10 if phone present)
capped at 100

company_size_points: 1-10→5, 11-50→15, 51-200→25, 201-500→35, 500+→40, unknown→10
source_points:       Website Form→25, LinkedIn Lead Gen→30, WhatsApp→20

temperature: hot ≥70, warm ≥40, else cold
```
