# FLOOWBOX - AI Workplace Bias Detector (Glassdoor + OpenAI)

Uses AI to surface potential demographic bias patterns in how employees describe their workplace experience. Built as a research tool for HR consultants.

## What it does

Scrapes Glassdoor reviews for a target company using ScrapingBee. Defines a dictionary of demographic keyword categories. Runs parallel OpenAI analysis chains — one per demographic dimension — to detect sentiment patterns and language differences across groups. Merges findings into a structured bias pattern report with QuickChart visualization.

## Tools Used
- **Orchestration:** n8n
- **Scraping:** ScrapingBee (Glassdoor)
- **Analysis:** Multiple parallel OpenAI GPT chains
- **Visualization:** QuickChart
- **Logic:** Demographic keyword dictionary, parallel branches, merge

## Why I built this
An HR research client needed to analyze workplace culture signals at scale across multiple companies. Manual review of 1000+ reviews per company wasn't feasible. This flags patterns worth investigating.
