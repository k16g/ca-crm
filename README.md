# CA Firm CRM — Intelligent Practice Management

A CRM and automation platform designed for Indian Chartered Accountancy (CA) firms to manage clients, compliance deadlines, document collection, leads, invoices, tasks, and AI-assisted client communication from one system.

The project contains two main components:

- **CRM Dashboard:** A browser-based HTML dashboard for client/practice management.
- **n8n Automation Backend:** A 152-node workflow that connects WhatsApp, Gmail, Google Sheets, Groq-powered AI agents, Telegram alerts, scheduled follow-ups, and webhook-driven events.

> **Current project structure:** the dashboard is a front-end/demo interface, while the n8n workflow contains the actual automation logic and external integrations.

---

## Short Description

**CA Firm CRM is an AI-powered practice-management CRM for Chartered Accountancy firms that automates client support, lead qualification, compliance reminders, document collection, invoice follow-ups, and internal alerts across WhatsApp and email.**

---

## Key Features

### 1. CRM Dashboard

The dashboard provides a centralized view of the firm's operations.

- Dashboard overview
- Client count and recent activity
- Pending compliance deadlines
- Fees collected and outstanding
- AI agent activity
- Lead pipeline overview
- Recent client activity
- Client management
- Task management
- Reports & analytics
- Settings/configuration area

### 2. AI Client Support Agent

The n8n workflow includes AI-powered support for existing clients.

**WhatsApp support**
- Receives incoming WhatsApp messages through the WhatsApp Cloud API.
- Identifies the client from their phone number.
- Loads relevant client information.
- Retrieves compliance and document context from Google Sheets.
- Maintains short-term conversation memory.
- Generates an AI response using Groq.
- Sends the response back through WhatsApp.
- Logs the interaction.

**Email support**
- Monitors incoming Gmail messages.
- Identifies existing clients by email address.
- Loads relevant compliance/document information.
- Uses an AI support agent to draft a response.
- Replies through Gmail.
- Logs the query.

### 3. WhatsApp Lead Qualification

New prospects can enter through WhatsApp.

The workflow can:

- Detect new prospects.
- Create/update a lead record.
- Maintain lead conversation memory.
- Ask qualification questions.
- Classify the lead using AI.
- Store qualification information in Google Sheets.
- Detect hot leads.
- Notify the CA/partner through Telegram when a hot lead is detected.

### 4. Website Lead Capture

A website can send leads to the CRM through:

`POST /webhook/website-lead`

The workflow:

1. Receives the lead.
2. Validates the payload.
3. Creates the lead in Google Sheets.
4. Generates an AI opening message.
5. Classifies the lead.
6. Updates the lead record.
7. Sends WhatsApp if a phone number is available.
8. Sends email if an email address is available.
9. Alerts the partner for urgent leads.

### 5. Document Collection & Tracking

The system tracks documents required for client work.

Examples include:

- PAN Card
- Aadhaar
- Bank Statement
- Balance Sheet
- P&L Statement
- GST Returns
- Purchase Bills
- Form 16
- Sales Register

Features include:

- Track required documents per client.
- Identify missing documents.
- Mark documents as received.
- Send WhatsApp reminders.
- Send email reminders.
- Schedule repeated follow-ups.
- Escalate persistent document delays to the partner/CA.

A dedicated webhook is also available for requesting documents programmatically:

`POST /webhook/request-documents`

### 6. Compliance Deadline Automation

The system tracks compliance items such as:

- GSTR-1
- GSTR-3B
- TDS returns
- ITR filing
- ROC annual returns
- Other firm-defined compliance activities

The n8n workflow runs a daily compliance check at **9:00 AM**.

It calculates the relevant reminder stage and can send:

- 7-day reminder
- 3-day reminder
- 1-day urgent reminder
- Due-today reminder
- Overdue reminder

Reminders can be delivered through:

- WhatsApp
- Email

Severely overdue items can also trigger a Telegram alert to the partner.

### 7. Invoice & Fee Follow-up

The CRM includes invoice tracking and automated collection follow-up.

The workflow can:

- Read invoices from Google Sheets.
- Calculate reminder stages.
- Generate AI-written payment reminders.
- Send WhatsApp reminders.
- Send email reminders.
- Track reminder counts.
- Detect severely overdue invoices.
- Escalate overdue accounts to the partner.

The daily invoice follow-up workflow runs at **10:30 AM**.

There is also a payment confirmation webhook:

`POST /webhook/payment-confirmation`

When a payment event is received, the workflow can:

- Find the invoice.
- Mark it as paid.
- Generate a thank-you message.
- Send WhatsApp confirmation.
- Send an email receipt.

### 8. Automated Lead Follow-up

The system runs a daily lead follow-up workflow at **10:00 AM**.

It can:

- Find leads requiring follow-up.
- Determine the next follow-up stage.
- Generate a personalized AI message.
- Send WhatsApp follow-up.
- Send email follow-up.
- Update the next follow-up date/status.

### 9. Automated Document Follow-up

The document follow-up workflow runs at **9:30 AM**.

It can:

- Find pending documents.
- Calculate whether a follow-up is required.
- Generate an AI-written reminder.
- Send WhatsApp reminders.
- Send email reminders.
- Update follow-up counts.
- Escalate repeated/persistent delays.

### 10. Partner / Internal Alerts

Telegram is used for internal notifications.

Alerts can be triggered for:

- Hot WhatsApp leads
- Urgent website leads
- Severely overdue compliance
- Document escalation
- Severely overdue invoices
- Workflow/system errors

### 11. Error Handling & Logging

The n8n workflow contains a workflow-level error trigger.

Errors can be:

- Captured and formatted.
- Logged to the `Error_Log` Google Sheet.
- Sent to the partner through Telegram.

WhatsApp queries and email queries are also logged for operational visibility.

---

# Architecture

```text
                    ┌──────────────────────┐
                    │    CA CRM Dashboard   │
                    │      HTML / CSS / JS  │
                    └──────────┬───────────┘
                               │
                               │
                    ┌──────────▼───────────┐
                    │         n8n           │
                    │ Automation Workflows  │
                    └──────────┬───────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          ▼                    ▼                    ▼
   WhatsApp Cloud API       Gmail              Website
          │                    │                 Leads
          │                    │                    │
          └────────────┬───────┴────────────────────┘
                       │
                       ▼
                ┌───────────────┐
                │  AI / Groq    │
                │ Llama 3.3 70B │
                └───────┬───────┘
                        │
          ┌─────────────┼─────────────┐
          │             │             │
          ▼             ▼             ▼
   Google Sheets     Telegram      WhatsApp
    CRM Database    Partner Alerts  Responses
```

---

# n8n Workflow Components

The supplied workflow contains approximately **152 nodes** covering:

### Communication

- WhatsApp incoming webhook
- WhatsApp replies
- Gmail incoming email
- Gmail replies
- WhatsApp document reminders
- Email document reminders
- WhatsApp compliance reminders
- Email compliance reminders
- WhatsApp invoice reminders
- Email invoice reminders

### AI

- Groq / OpenAI-compatible chat model
- WhatsApp support agent
- Email support agent
- WhatsApp lead qualification agent
- Lead classification
- Compliance reminder generation
- Document reminder generation
- Lead follow-up generation
- Invoice reminder generation
- Payment thank-you generation

### Data

Google Sheets is used as the operational data store.

The workflow references these sheets:

- `Clients`
- `Leads`
- `Documents_Tracker`
- `Compliance_Calendar`
- `Invoices`
- `Query_Log`
- `Error_Log`

### Automation

Scheduled jobs include:

| Workflow | Schedule |
|---|---:|
| Compliance check | Daily 9:00 AM |
| Document follow-up | Daily 9:30 AM |
| Lead follow-up | Daily 10:00 AM |
| Invoice follow-up | Daily 10:30 AM |

---

# Technology Stack

## Frontend

- HTML5
- CSS3
- Vanilla JavaScript
- Inter font
- Space Grotesk font
- Responsive browser-based dashboard

## Automation

- n8n
- n8n Webhooks
- n8n Schedule Triggers
- n8n Code nodes
- n8n IF/conditional nodes
- n8n LangChain nodes

## AI

- Groq API
- OpenAI-compatible API interface
- `llama-3.3-70b-versatile`
- n8n AI Agent nodes
- n8n conversation memory

## Integrations

- Meta WhatsApp Cloud API
- Gmail
- Google Sheets
- Telegram Bot API

---

# Project Files

```text
CA Crm/
├── ca-crm (1).html
├── n8n.json
└── README.md
```

### `ca-crm (1).html`

The CRM dashboard frontend.

It currently contains the UI for:

- Dashboard
- AI Support Agent
- Document Hub
- Compliance Tracker
- Lead Pipeline
- Invoice & Fees
- Client Management
- Tasks
- Reports
- Settings

The frontend currently operates as a browser-side dashboard/demo. Its displayed data is embedded in the HTML rather than being dynamically fetched from the n8n workflow.

### `n8n.json`

The complete n8n automation workflow containing the backend integrations and automation logic.

---

# Requirements

## Required

### 1. n8n

Install or host an n8n instance.

Recommended for production:

- n8n Cloud, or
- Self-hosted n8n

### 2. Google Account

Required for:

- Google Sheets
- Gmail

You need OAuth credentials with the required permissions.

### 3. Google Sheets Database

Create a Google Spreadsheet and configure the workflow to use it.

The workflow expects these sheets:

```text
Clients
Leads
Documents_Tracker
Compliance_Calendar
Invoices
Query_Log
Error_Log
```

The exact column structure should match the fields used by the imported n8n nodes.

### 4. Groq API Key

Create a Groq API key and connect it to the n8n Groq/OpenAI-compatible credential.

The supplied workflow uses:

```text
llama-3.3-70b-versatile
```

### 5. Meta WhatsApp Cloud API

Create/configure a Meta WhatsApp Business application.

You need:

- WhatsApp Business Account
- Phone Number ID
- Access token
- Webhook configuration
- Webhook verification token

The workflow sends messages through the Meta Graph API.

### 6. Gmail

Connect a Gmail account through n8n OAuth.

The workflow uses Gmail for:

- Incoming client email
- Email support replies
- New lead responses
- Document reminders
- Compliance reminders
- Invoice reminders
- Payment confirmations/receipts

### 7. Telegram Bot

Create a Telegram bot and connect it to n8n.

It is used for internal partner/CA alerts.

---

# Configuration

The supplied n8n workflow contains placeholders that must be replaced/configured before production use.

Important placeholders include:

```text
YOUR_CA_FIRM_NAME_PLACEHOLDER
WHATSAPP_PHONE_NUMBER_ID_PLACEHOLDER
WHATSAPP_WEBHOOK_VERIFY_TOKEN_PLACEHOLDER
TELEGRAM_PARTNER_CHAT_ID_PLACEHOLDER
```

The Google Sheets document is also represented by a placeholder/reference:

```text
GOOGLE_SHEET_ID_CA_FIRM_DB
```

Credential references in the exported workflow include:

```text
Gmail - CA Firm
Google Sheets - CA Firm
WhatsApp Cloud API Token
Groq API Key (OpenAI Compatible)
Telegram - Partner Alerts Bot
```

When importing the workflow into another n8n instance, reconnect these credentials to the corresponding credentials in the new instance.

---

# Setup Guide

## Step 1 — Open the Dashboard

The frontend does not require a build process.

Simply open:

```text
ca-crm (1).html
```

in a modern browser.

For a local server, you can also use:

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

---

## Step 2 — Set Up n8n

Import:

```text
n8n.json
```

into your n8n instance.

After importing, review all credentials and reconnect them to your own accounts.

---

## Step 3 — Configure Google Sheets

Create the CRM spreadsheet with:

```text
Clients
Leads
Documents_Tracker
Compliance_Calendar
Invoices
Query_Log
Error_Log
```

Update the Google Sheets document reference in the workflow.

Make sure the sheet headers match the fields expected by the n8n Code and Google Sheets nodes.

---

## Step 4 — Configure Groq

Create a Groq API credential in n8n.

Connect it to the AI model node.

The workflow is configured around:

```text
llama-3.3-70b-versatile
```

---

## Step 5 — Configure WhatsApp

In Meta Developer / WhatsApp Cloud API:

1. Create or use a WhatsApp Business application.
2. Obtain the Phone Number ID.
3. Generate/configure the access token.
4. Configure the n8n webhook.
5. Set the webhook verification token.
6. Subscribe to the relevant WhatsApp message events.
7. Replace the Phone Number ID placeholder in the HTTP request nodes.

The incoming webhook path in the workflow is:

```text
/webhook/whatsapp-incoming
```

The workflow supports both Meta webhook verification and incoming message processing.

---

## Step 6 — Configure Gmail

Connect Gmail through n8n OAuth.

The workflow includes Gmail trigger/send nodes for client support and automated communication.

Confirm that the Gmail account has the appropriate permissions before activating the workflow.

---

## Step 7 — Configure Telegram

Create a Telegram bot using BotFather.

Connect the Telegram API credential to n8n.

Set the partner/CA chat ID where internal alerts should be delivered.

Replace:

```text
TELEGRAM_PARTNER_CHAT_ID_PLACEHOLDER
```

with the appropriate destination.

---

## Step 8 — Configure Website Lead Form

Your website can POST lead information to:

```text
/webhook/website-lead
```

The exact JSON payload should match the fields expected by the `Validate & Parse Lead Payload` node.

Typical lead information includes:

```json
{
  "name": "Example Client",
  "phone": "+91XXXXXXXXXX",
  "email": "client@example.com",
  "requirement": "GST Registration"
}
```

---

## Step 9 — Configure Document Requests

External systems can send document requests to:

```text
/webhook/request-documents
```

The workflow will locate the client, create document tracker entries, generate a message, and send the request through the available communication channel.

---

## Step 10 — Configure Payment Confirmation

A payment provider or other external system can send payment confirmation to:

```text
/webhook/payment-confirmation
```

The workflow matches the invoice, marks it as paid, and sends a confirmation/thank-you message.

---

# Scheduled Automation

Once the workflows are active, n8n automatically runs the scheduled jobs.

### 9:00 AM — Compliance

Checks upcoming/overdue compliance records and sends appropriate reminders.

### 9:30 AM — Documents

Checks missing documents and sends follow-ups.

### 10:00 AM — Leads

Checks leads requiring follow-up and sends AI-generated messages.

### 10:30 AM — Invoices

Checks outstanding invoices and sends payment reminders.

The schedule uses the timezone configured for the n8n instance. Configure the instance timezone appropriately for the firm's operating location.

---

# Production Checklist

Before going live, verify:

- [ ] n8n is running reliably.
- [ ] All n8n credentials are connected.
- [ ] Google Sheets database exists.
- [ ] All required sheet tabs exist.
- [ ] Google Sheets headers match the workflow.
- [ ] Groq API key is configured.
- [ ] WhatsApp Cloud API is configured.
- [ ] WhatsApp webhook is publicly reachable over HTTPS.
- [ ] WhatsApp verification token is configured.
- [ ] WhatsApp Phone Number ID is configured.
- [ ] Gmail OAuth is connected.
- [ ] Telegram bot is configured.
- [ ] Telegram partner chat ID is configured.
- [ ] Firm name placeholders have been replaced.
- [ ] Scheduled workflows are enabled.
- [ ] Webhook workflows are enabled.
- [ ] Test client data has been added.
- [ ] Test WhatsApp message has been processed.
- [ ] Test email has been processed.
- [ ] Test lead has been created.
- [ ] Test document reminder has been sent.
- [ ] Test compliance reminder has been sent.
- [ ] Test invoice reminder has been sent.
- [ ] Test payment confirmation has been processed.
- [ ] Error handling has been tested.

---

# Security Considerations

This system processes potentially sensitive client, financial, tax, and communication information.

For production deployment:

- Never commit API keys or access tokens to Git.
- Use n8n credentials rather than hardcoding secrets.
- Use HTTPS for all public webhooks.
- Restrict Google Sheets permissions.
- Use a dedicated business Google account.
- Restrict Telegram alerts to authorized recipients.
- Validate all incoming webhook payloads.
- Add authentication/signatures to public webhooks where appropriate.
- Avoid exposing client data in frontend source code.
- Implement proper user authentication before connecting the dashboard to real client data.
- Maintain backups of the Google Sheets database.
- Review data retention and access policies before production use.

---

# Important Current Limitations

The supplied project is best understood as a **working CRM concept/prototype plus an automation backend**, rather than a fully integrated production SaaS application.

### Dashboard/backend connection

The HTML dashboard currently contains static/demo data and browser-side interactions. It does not appear to directly fetch live data from n8n or Google Sheets.

A production version should add an API/backend layer so that dashboard metrics, clients, invoices, documents, leads, tasks, and reports come from live data.

### Authentication

The supplied HTML dashboard does not implement production-grade authentication or authorization.

A production deployment should add:

- Login
- Role-based access control
- Session management
- Admin/user management
- Audit logging

### Database

Google Sheets is being used as the operational database in the supplied workflow. This is convenient for an MVP, but a production system may benefit from PostgreSQL or another proper database as data volume and concurrency increase.

### Client document storage

The workflow tracks documents, but the supplied project does not provide a complete secure document-storage portal comparable to a production client portal.

### Payment gateway

The workflow has a payment confirmation webhook, but a specific payment provider integration is not included in the supplied files.

---

# Suggested Production Architecture

For a production SaaS implementation, the architecture could evolve into:

```text
                 ┌────────────────────┐
                 │   Web CRM / Portal │
                 │ React / Next.js    │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │    API / Backend   │
                 │ FastAPI / Node.js  │
                 └─────────┬──────────┘
                           │
            ┌──────────────┼──────────────┐
            ▼              ▼              ▼
       PostgreSQL      Object Storage     n8n
        Database       Documents        Automation
                                           │
                    ┌──────────────────────┼──────────────────┐
                    ▼                      ▼                  ▼
                 WhatsApp               Gmail              Groq
                    │                      │                  │
                    └──────────────────────┼──────────────────┘
                                           ▼
                                      AI Automation
```

This would allow the existing n8n automation layer to remain useful while replacing the static dashboard and Google Sheets storage with a scalable application backend.

---

# Webhook Summary

| Purpose | Method | Path |
|---|---|---|
| WhatsApp incoming messages | GET / POST | `/webhook/whatsapp-incoming` |
| Website leads | POST | `/webhook/website-lead` |
| Document requests | POST | `/webhook/request-documents` |
| Payment confirmations | POST | `/webhook/payment-confirmation` |

The exact production webhook URLs depend on the n8n deployment and webhook configuration.

---

# Automation Summary

```text
WhatsApp Client
      │
      ▼
Identify Client
      │
      ├── Existing Client ──► Load Context ──► AI Support ──► WhatsApp Reply
      │
      └── New Lead ─────────► Qualification ─► Classification
                                      │
                                      └── Hot Lead ─► Telegram Alert


Website Lead
      │
      ▼
Validate
      │
      ▼
Create Lead
      │
      ▼
AI Classification
      │
      ├── WhatsApp
      ├── Email
      └── Urgent ──► Telegram


Daily Schedules
      │
      ├── 09:00 ──► Compliance Reminders
      ├── 09:30 ──► Document Follow-ups
      ├── 10:00 ──► Lead Follow-ups
      └── 10:30 ──► Invoice Follow-ups
```

---

# License

No license file was included with the supplied project.

Before publishing or distributing the project, add an appropriate license and clarify ownership of the dashboard code, n8n workflow, prompts, branding, and any third-party integrations.

---

# Status

**Project type:** AI-powered CA practice-management CRM / automation prototype

**Frontend:** HTML/CSS/JavaScript dashboard

**Automation:** n8n

**AI:** Groq + Llama 3.3 70B

**Data:** Google Sheets

**Messaging:** WhatsApp Cloud API + Gmail

**Internal alerts:** Telegram

**Automation workflows:** 152 n8n nodes

