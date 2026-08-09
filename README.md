# Med Spa Outreach Automation

A set of four n8n workflows that together cover lead generation and cold
outreach for med spa businesses: spotting buying/hiring signals, finding
direct-contact leads from directory search, sending AI-personalized cold
email sequences, and detecting replies. Each workflow runs independently
on its own trigger (schedule or manual run) rather than being chained via
webhooks, and they read/write to two Google Sheets that act as the shared
state between them.

## Workflows in this repo

### `signal-scanner-google-alerts.json` — Signal Scanner
Runs daily. Polls a set of Google Alerts RSS feeds (e.g. "new med spa
Austin", "med spa front desk hiring Nashville") for buying/hiring signals
tagged by city and signal type, parses each feed's articles, and logs new
signals to a "Signals" sheet.

### `instant-lead-finder-directory-search.json` — Instant Lead Finder
Run on demand. For a list of cities and med-spa-related search templates,
searches Google (via the Serper API), filters out social/directory/review
sites to keep only independent business websites, visits each site to
extract a contact email and best-guess person/company name, and appends
new (non-duplicate) leads to a "Raw leads" sheet.

### `cold-email-outreach-sender.json` — Cold Email Sender (AI Personalized)
Runs weekdays at 9am. Reads the leads sheet, decides each lead's next
outreach step from its current status (initial email, 1st/2nd/3rd
follow-up) and how long it's been since last contact, fetches the
company's website for context, and — for a first email — uses an LLM to
diagnose one realistic, specific operational pain point for that business
and draft a personalized cold email around it. Sends the appropriate email
via SMTP and updates the lead's status.

### `cold-email-reply-checker.json` — Reply Checker
Runs every 2 hours. Checks an IMAP inbox for recent messages, matches
senders against the leads sheet by email address, and marks matching leads
as "Replied" so the sender workflow stops following up with them.

## Tools & APIs

- [n8n](https://n8n.io) — workflow orchestration
- [Serper](https://serper.dev) — Google Search API
- Google Sheets API — lead storage and status tracking
- IMAP — inbox monitoring for replies
- SMTP — sending outreach emails
- OpenAI (gpt-4o-mini) — personalized pain-point diagnosis and email copy

## Setup

These exports have all credentials, API keys, resource IDs (spreadsheets),
Google Alerts feed URLs, and personal sender info removed. To run this
yourself:

1. Import all four JSON files into your n8n instance.
2. Create your own Serper, OpenAI, Google Sheets, IMAP, and SMTP
   credentials in n8n.
3. Replace the `YOUR_..._HERE` placeholders in each workflow: your own
   Google Alerts feed URLs (Signal Scanner), spreadsheet IDs (all four),
   and your sender name/email/booking link (Cold Email Sender).
4. Point the two Google Sheets resource locators at your own "Leads" and
   "Raw leads" spreadsheets, matching the column names used in each
   workflow's Google Sheets nodes.
5. Activate the scheduled workflows, and run Instant Lead Finder manually
   whenever you want to source new leads for a city.
