# research-report-generator

Research takes time. Finding relevant sources, reading them, synthesizing across them, pulling out the data points, organizing the findings, writing it up — a decent research brief takes half a day. Most of that time is mechanical.

This runs two parallel Tavily searches (primary research + data/statistics), synthesizes findings across up to 13 sources, and produces a structured research report with executive summary, key findings with implications, data highlights, full narrative sections, opportunities and risks, conclusions, recommended next steps, knowledge gaps, and a source list. The whole thing takes under 2 minutes.

Works for market analysis, competitive landscapes, industry overviews, technology assessments, investment theses, trend reports, and more.

---

## What it does

1. Accepts a POST: topic, report type, audience, specific questions to answer, scope, geographic focus, timeframe
2. Runs two parallel Tavily searches: primary research (advanced depth, 8 results) + data/statistics (basic depth, 5 results)
3. Claude synthesizes all sources and writes:
   - Report title, executive summary (3–5 sentences)
   - Key findings with evidence and implications
   - Data highlights (specific statistics with context)
   - Full narrative sections calibrated to the report type
   - Opportunities and risks/challenges
   - Conclusions, recommended next steps, knowledge gaps
   - Source list with URLs
4. Builds a professionally formatted HTML report
5. Emails if `reply_email` provided
6. Returns full JSON

---

## Stack

- **n8n** — webhook + workflow
- **Tavily API** — two parallel searches
- **Anthropic Claude** (claude-sonnet-4-20250514) — report writing
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=research@yourcompany.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-report \
  -H "Content-Type: application/json" \
  -d '{
    "topic": "AI-powered customer service automation",
    "report_type": "market_analysis",
    "audience": "executive",
    "geographic_focus": "North America",
    "timeframe": "2024-2026",
    "scope": "Focus on mid-market B2B SaaS companies adopting AI for first-line customer support.",
    "specific_questions": [
      "What is the current market size and growth rate?",
      "Who are the leading vendors?",
      "What are the primary barriers to adoption?"
    ],
    "reply_email": "strategy@company.com"
  }'
```

**Required:** `topic`, `report_type`

---

## Report types

`market_analysis`, `competitive_landscape`, `industry_overview`, `technology_assessment`, `investment_thesis`, `due_diligence`, `trend_report`, `feasibility_study`, `background_brief`

---

## Audience calibration

`executive`, `investor`, `technical`, `general_business`, `board` — each shapes tone and emphasis.

---

## Specific questions

Pass up to 8 questions to shape the report directly. Specific questions produce specific reports.

---

## Limitations

- Research quality depends on what Tavily finds — paywalled reports (Gartner, IDC) won't be fully accessible.
- For external publication, verify key statistics against primary sources.
- Very niche topics may return limited results — the report is honest about gaps.

---

## Ideas

- [ ] PDF export
- [ ] Report library in Google Drive
- [ ] Scheduled tracking — same topic monthly
- [ ] Multi-topic comparison reports

---

## License

MIT.
