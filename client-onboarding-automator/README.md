# client-onboarding-automator

New client onboarding is the thing everyone says is important and then handles inconsistently. Someone manually writes a welcome email, the account manager gets a Slack message, and two weeks later you realize the kickoff agenda was never prepared. When things go wrong in the first 90 days, it's usually because the first week was chaotic.

This takes a new client record, generates all the onboarding materials in one shot, sends the welcome email to the client, sends the internal handoff note to the account manager, pings Slack, and logs the client to a Google Sheet. The whole flow runs in under 30 seconds after the webhook fires.

Claude writes the materials tailored to the service type — a welcome email for a retained law client reads differently from one for a SaaS implementation. The checklist and kickoff agenda are structured for the specific engagement, not pulled from a generic template.

---

## What it does

1. POST a new client: name, email, company, service type, package tier, account manager, start date, contract value, project details
2. Saves client record to Google Sheets (CRM log)
3. Claude generates:
   - Welcome email (to client) — confirms the engagement, what happens next, how to reach the team
   - Internal handoff note (to account manager) — who the client is, key context, what week 1 looks like
   - Onboarding checklist — phased tasks by owner (AM / client / both) across Week 1, 2, Month 1, Ongoing
   - Kickoff meeting agenda — timed agenda items with purpose
   - Success metrics — what good looks like after 30/60/90 days
   - Red flags — early warning signs to watch for
4. Sends welcome email to client
5. If `account_manager_email` provided: sends internal handoff + checklist
6. If `notify_slack_channel` provided: posts new client notification
7. Returns all generated materials as JSON

---

## Stack

- **n8n** — webhook + workflow
- **Google Sheets** — client CRM log
- **Anthropic Claude** (claude-sonnet-4-20250514) — onboarding materials
- **SMTP** — email delivery
- **Slack** (optional) — team notification

---

## Setup

### 1. Create the Google Sheet

One tab: **Clients** — columns:
```
client_id | client_name | client_email | client_company | service_type | package_tier | account_manager | start_date | contract_value | currency | status | created_at
```

### 2. Environment variables

```
CLIENTS_SHEET_ID=your_google_sheet_id
FROM_EMAIL=onboarding@yourcompany.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**
- **Slack API** (optional)

### 4. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/onboard-client \
  -H "Content-Type: application/json" \
  -d '{
    "client_name": "Sarah Okafor",
    "client_email": "sarah@growthco.io",
    "client_company": "GrowthCo",
    "client_phone": "+1 415 555 0192",
    "service_type": "SEO retainer",
    "package_tier": "growth",
    "account_manager": "James Rivera",
    "account_manager_email": "james@youragency.com",
    "start_date": "2025-05-12",
    "contract_value": 3500,
    "currency": "USD",
    "project_details": "6-month SEO retainer focused on growing organic traffic for their SaaS product. Primary goal is ranking for 15 target keywords in the B2B project management space.",
    "specific_needs": "Client has existing blog content that needs optimization before we start publishing new. They also want monthly reporting calls on the 1st of each month.",
    "notify_slack_channel": "#new-clients"
  }'
```

**Required:** `client_name`, `client_email`, `service_type`, `account_manager`

---

## What gets generated

**Welcome email** — sent directly to the client. Warm, professional, specific to what they've signed up for. Confirms the next step and gives them a direct contact. Under 200 words — long enough to feel personal, short enough to actually get read.

**Internal handoff note** — sent to the account manager. Summarizes who the client is, what they care about most, and what the first week should prioritize. Includes the onboarding checklist and kickoff agenda inline.

**Onboarding checklist** — phased tasks with owner and notes. Week 1 covers the immediate setup and intro steps; Month 1 covers deeper integration; Ongoing covers rhythmic touchpoints. The tasks are tailored to the service type.

**Kickoff agenda** — timed agenda with purpose for each item. Ready to paste into a calendar invite or doc.

**Success metrics** — what good looks like. Specific to the service. Useful to share with the client at kickoff so expectations are aligned from day one.

**Red flags** — early warning signs that onboarding is going off track. Things like: client not responding to onboarding questionnaire after a week, unclear ownership on the client side, scope being pushed before foundations are set.

---

## Triggering from your existing tools

The webhook fires from anything that can make a POST request:

- **CRM webhook** (HubSpot, Pipedrive, Salesforce): trigger on deal closed-won
- **Form submission** (Tally, Typeform): fire when a client completes a signup form
- **Manual** from n8n: use the Test Webhook and fill in the data
- **Google Sheets trigger**: another n8n workflow watches a sheet for new rows

For the CRM trigger approach, you'd add a node before Validate Input that maps the CRM payload fields to the expected body format.

---

## Package tiers

The `package_tier` field is freeform but shapes Claude's framing of the checklist and materials. Common values: `starter`, `standard`, `growth`, `enterprise`, `custom`. A higher-tier client typically gets more touchpoints in the checklist.

---

## Multiple service types

Claude handles a wide range of service types and calibrates accordingly. Examples that work well:
- `SEO retainer`, `paid ads management`, `content marketing`
- `software development`, `UX design`, `IT support`
- `accounting`, `legal services`, `financial planning`
- `coaching`, `consulting`, `training`
- `property management`, `facilities management`

The more specific the `service_type` value, the more specific the generated materials.

---

## Limitations

- The welcome email is sent directly from the workflow — make sure your `FROM_EMAIL` domain has proper SPF/DKIM to avoid spam filters, especially for client-facing sends.
- Generated materials are a starting point. Review the welcome email before trusting it for a high-value client — occasionally Claude uses a phrase that doesn't match your brand voice.
- No contract or document generation. If you want to auto-generate a scope of work or contract, add a document generation step using the docx skill or a PDF node.

---

## Ideas

- [ ] 30/60/90 day check-in scheduler: automatically create calendar events for milestone check-ins
- [ ] Client portal creation: trigger provisioning of a client Notion space or Google Drive folder
- [ ] NPS survey trigger: schedule an automated satisfaction check at 30 days
- [ ] Off-boarding mirror: a companion workflow for when clients churn or contracts end

---

## License

MIT.
