# FLOOWBOX - Conversion Rate Optimizer Agent

CRO consultants charge ₹50,000+ for a landing page audit. This workflow does the same analysis in 90 seconds — reading your actual page content, benchmarking against industry data, and producing specific actionable fixes with effort estimates.

## What it does

Send any page URL, your current conversion rate, and industry. Jina AI fetches the full page content. Perplexity researches current industry benchmarks and 2025-2026 CRO best practices for that page type. GPT-4o audits the page across 8 CRO dimensions — headline clarity, CTA strength and placement, trust signals, page friction, mobile optimization, above-the-fold content, form design, and urgency elements. Returns critical issues with specific fixes, quick wins, rewritten headline and CTA suggestions, and an estimated CVR improvement.

## Tools Used
- **Orchestration:** n8n
- **Page Fetching:** Jina AI
- **Benchmark Research:** Perplexity AI Sonar
- **CRO Audit:** OpenAI GPT-4o
- **Storage:** Notion
- **Report:** Slack
- **Trigger:** Webhook

## 8 audit dimensions

| Dimension | What gets checked |
|---|---|
| Headline | Clarity, value proposition, matches audience intent |
| CTA | Wording strength, placement, visibility, color contrast |
| Trust Signals | Reviews, badges, guarantees, logos |
| Friction | Number of steps, form fields, required info |
| Mobile | Mobile-friendly layout signals |
| Above the Fold | Key message visible without scrolling |
| Form Design | Fields, labels, error handling |
| Urgency | Scarcity, countdown, limited availability |

## Audit output

```json
{
  "current_cvr": 2.4,
  "industry_benchmark": "3.5-5% for SaaS landing pages",
  "critical_issues": [
    {"issue": "CTA says 'Submit' — lowest converting CTA word", "impact": "high", "fix": "Change to 'Start Free Trial' or 'Get Instant Access'", "effort": "quick"},
    {"issue": "No testimonials above the fold", "impact": "high", "fix": "Move 3 star reviews to hero section", "effort": "medium"}
  ],
  "headline_analysis": {
    "current": "The Best Automation Tool",
    "problem": "Generic — no specific benefit, no target audience",
    "suggested": "Automate Your Business in a Weekend — No Code Required"
  },
  "quick_wins": ["Add SSL badge near signup button", "Change 'Submit' → 'Get Started Free'"],
  "estimated_cvr_improvement": "Fixing critical issues alone could increase CVR by 40-60% to reach industry benchmark"
}
```

## Why I built this

A FLOOWBOX client's landing page had a 1.8% CVR on paid traffic. Running this audit flagged 4 critical issues — all fixable in one afternoon. After implementing: CVR went to 3.2% — same ad spend, 78% more signups.

## Webhook payload

```json
{
  "url": "https://your-landing-page.com",
  "page_type": "landing page",
  "current_cvr": 2.4,
  "industry": "SaaS",
  "goal": "free trial signup"
}
```

## Setup

1. Jina AI API key
2. Perplexity API key
3. OpenAI API key
4. Notion integration + DB ID
5. Slack Bot Token + #analytics channel
