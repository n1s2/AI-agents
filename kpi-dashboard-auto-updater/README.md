# FLOOWBOX - KPI Dashboard Auto-Updater

The morning standup should start with everyone already knowing the key numbers. This workflow computes all critical KPIs from raw data, adds a GPT-4o narrative with one recommended action, and posts everything to Slack before the team even opens their laptops.

## What it does

Every morning at 8 AM, pulls the latest data from three Google Sheets — daily revenue, customer metrics, and product sales. Calculates day-over-day revenue change, week-to-date and month-to-date totals, new customers, churn, and top product by units. GPT-4o writes a 2-3 sentence business narrative with context and one specific action item. Updates the dashboard log sheet and posts to #daily-standup Slack channel.

## Tools Used
- **Orchestration:** n8n
- **Data (x3):** Google Sheets (parallel fetch)
- **Computation:** Code node (DoD, WTD, MTD calculations)
- **Narrative:** OpenAI GPT-4o
- **Dashboard:** Google Sheets (log)
- **Standup:** Slack
- **Schedule:** Daily 8 AM

## Example daily standup output

```
📊 Daily KPI Update — 2026-04-17
Status: GOOD

Revenue of ₹28,400 today is 12% above yesterday — driven by the weekend traffic 
tail carrying into Monday morning. New customer acquisitions (4) are on pace for 
the weekly target. Churn of 1 is within normal range.

Today: ₹28,400 (+12% DoD)
WTD: ₹142,000
MTD: ₹380,000

New customers: 4 | Churned: 1
Top product: Automation Starter Kit (8 units)

Today's action: Follow up with 3 enterprise leads who opened the proposal email last week.
```

## Google Sheets schema

**Daily Revenue:** Date, Revenue
**Daily Customers:** Date, New Customers, Active Customers, Churned
**Daily Products:** Date, Product, Units Sold, Revenue

## Why I built this

Before this workflow, the FLOOWBOX team started every day not knowing whether yesterday was good or bad. Opening dashboards, aggregating numbers, and writing a summary took 20 minutes. Now the summary is already in Slack when the day starts — and the action item keeps everyone focused on the right thing.

## Setup

1. Google Sheets with the three data sheets
2. OpenAI API key
3. Slack Bot Token + #daily-standup channel
