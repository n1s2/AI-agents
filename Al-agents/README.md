# 🚀 AI Agent Workflows — FLOOWBOX

Hi, I'm Navtej — founder of FLOOWBOX. This repo documents the AI agents and automation workflows I've built while running FLOOWBOX and working with clients.

My goal: bridge the gap between AI research and real business implementation. Every workflow here started as a problem I faced — either at FLOOWBOX or while helping a client automate something they were doing manually.

## 🛠️ Stack

| Tool | Purpose |
|---|---|
| n8n | Workflow orchestration |
| OpenAI GPT-4o / o3 | Reasoning, extraction, generation |
| Google Gemini | Search re-ranking, summarization |
| DeepSeek | Cost-efficient data extraction |
| Mistral | Lightweight LLM tasks |
| Qdrant | Vector search and RAG |
| Perplexity AI | Real-time web-grounded answers |
| SerpAPI + Brave | Web search |
| Apify + ScrapingBee + Jina AI | Advanced scraping |
| Baserow / NocoDB / Airtable | Data storage |

---

## 📂 Workflows

### 🔍 Research & Intelligence

**[Open Deep Research Agent](./Open_Deep_Research_-_AI-Powered_Autonomous_Research_Workflow.json)**
Full autonomous research pipeline — query in, structured report out. Uses SerpAPI + Jina AI + Gemini. My alternative to paid research tools.

**[Self-Hosted Deep Research Agent (Apify + OpenAI o3)](./Host_Your_Own_AI_Deep_Research_Agent_with_n8n__Apify_and_OpenAI_o3.json)**
Heavier version using Apify for JS-heavy pages and o3 for deep reasoning. Built for research tasks that need login-protected or SPA scraping.

**[Semantic Search Re-Ranking (Brave + Gemini)](./Intelligent_Web_Query_and_Semantic_Re-Ranking_Flow_using_Brave_and_Google_Gemini.json)**
Brave Search results re-ranked by Gemini for semantic relevance. Raw keyword ranking is rarely what you actually want.

**[Perplexity AI Integration](./Query_Perplexity_AI_from_your_n8n_workflows.json)**
Drop-in Perplexity API node for any n8n workflow. Real-time web-grounded answers in one call — no scrape stack needed.

---

### 🌐 Scraping & Content

**[Ultimate JS-Rendered Site Scraper (Selenium)](./Ultimate_Scraper_Workflow_for_n8n.json)**
Full browser session management for sites that block HTTP scrapers. Built for a client whose target site was a JavaScript SPA.

**[Bulk Webpage Scraper + AI Summarizer](./Scrape_and_summarize_webpages_with_AI.json)**
Fetch any list of URLs and get clean AI summaries back. I use a modified version for competitor content monitoring.

**[News Site Scraper Without RSS → NocoDB](./Scrape_and_summarize_posts_of_a_news_site_without_RSS_feed_using_AI_and_save_them_to_a_NocoDB.json)**
Scrapes industry news sites with no RSS feed. Filters last 7 days, generates summaries + keywords, saves to NocoDB.

**[Trustpilot Scraper + Sentiment Analyzer (DeepSeek + OpenAI)](./Scrape_Trustpilot_Reviews_with_DeepSeek__Analyze_Sentiment_with_OpenAI.json)**
Brand reputation monitoring. DeepSeek extracts structured review data, OpenAI runs sentiment analysis.

---

### 📊 Analytics & Reporting

**[Google Analytics AI Weekly Report → Baserow](./Send_Google_analytics_data_to_A_I__to_analyze_then_save_results_in_Baserow.json)**
Automated Monday morning GA4 reports. This week vs last week, analyzed by LLM, saved to Baserow.

**[SERPBear SEO Rankings → AI Brief → Baserow](./Summarize_SERPBear_data_with_AI__via_Openrouter__and_save_it_to_Baserow.json)**
Keyword ranking data from my self-hosted SERPBear instance turned into a weekly SEO brief by LLM.

**[Umami Analytics Week-over-Week → Baserow](./Summarize_Umami_data_with_AI__via_Openrouter__and_save_it_to_Baserow.json)**
Privacy-first analytics (Umami) with AI comparison and content suggestions. Runs every Thursday.

---

### 🤖 AI Agents & RAG

**[RAG with Proper OpenAI Citation Formatting](./Make_OpenAI_Citation_for_File_Retrieval_RAG.json)**
Fixes the ugly default `【4:0†source】` OpenAI citation markers. Replaces them with clean `_(filename)_` references for any client-facing RAG system.

**[Semantic Recipe Recommendations (Qdrant + Mistral)](./Recipe_Recommendations_with_Qdrant_and_Mistral.json)**
Built to learn Qdrant properly — full loop from embedding to semantic search to generation.

**[Survey Response Analyzer (Qdrant + Python)](./Survey_Insights_with_Qdrant__Python_and_Information_Extractor.json)**
Converts raw survey CSVs into semantically searchable Q&A pairs in Qdrant. Built for a 200-response customer survey that needed pattern analysis.

**[AI Rent Reconciliation Agent (Excel + OpenAI)](./Reconcile_Rent_Payments_with_Local_Excel_Spreadsheet_and_OpenAI.json)**
Proper agentic workflow. Watches for bank statement CSVs, reconciles against tenant Excel sheet using GPT-4o, flags every discrepancy. Saves a property manager client 4+ hours/month.

---

### 📰 Hacker News Workflows

**[HN Job Market Intelligence Scraper](./Hacker_News_Job_Listing_Scraper_and_Parser.json)**
Parses "Who is Hiring" threads into structured data. Filter by stack, location, remote — without reading 500 comments.

**[Learn Anything via HN Recommendations](./Learn_Anything_from_HN_-_Get_Top_Resource_Recommendations_from_Hacker_News.json)**
Submit a topic via form → get top learning resources from HN comments delivered to your email.

**[HN Stories to Video Scripts](./Hacker_News_to_Video_Content.json)**
Trending HN stories converted to video scripts + assets uploaded to Minio. Generates a week of content in one run.

---

### 🔎 Business Intelligence

**[AI Workplace Bias Detector (Glassdoor + OpenAI)](./Spot_Workplace_Discrimination_Patterns_with_AI.json)**
Scrapes Glassdoor reviews, runs parallel OpenAI chains to surface demographic sentiment patterns. Built for HR research use cases.

---


### 🏠 Property & Real Estate

**[Property Inventory Enrichment Agent (AI Vision + Web Search)](./Enrich_Property_Inventory_Survey_with_Image_Recognition_and_AI_Agent.json)**
Photos from property surveys → structured inventory data with market pricing. OpenAI Vision + Reverse Image Search + Firecrawl. Replaces manual item tagging for a property management client.

---

### 📺 Content Intelligence

**[YouTube Channel Intelligence Agent](./Extract_insights___analyse_YouTube_comments_via_AI_Agent_chat.json)**
Chat with any YouTube channel. Ask about comment sentiment, top videos, audience pain points, transcripts. Multi-turn sessions with Postgres memory.

**[AI SEO Seed Keyword Generator](./Generate_SEO_Seed_Keywords_Using_AI.json)**
Input your ICP → get 15-20 targeted seed keywords in 2 minutes. Claude Sonnet analyzes your product, pain points, goals, and customer expertise level. ~$0.02 per run.

## 📅 Status — March 2026

Active work:
- Client onboarding automation pipelines for FLOOWBOX
- WhatsApp lead capture + CRM sync (dropping this week)
- RAG-based customer support agents
- MS in AI applications — Fall 2027 intake

---

## 💡 About FLOOWBOX

FLOOWBOX builds custom AI agents and automation workflows for businesses. We connect LLMs with real business data — turning manual operations into intelligent pipelines.

*All workflows built and tested on self-hosted n8n. Total: 22 workflows.*
