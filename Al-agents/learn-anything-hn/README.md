# FLOOWBOX - Learn Anything via HN Community Recommendations

HN "Ask HN: How do I learn X?" threads contain some of the best curated resource lists on the internet. This workflow makes them accessible automatically.

## What it does

User fills a form with a topic and their email. The workflow searches HN "Ask HN" threads related to that topic, fetches all comments, passes them to Gemini which identifies and categorizes the best learning resources by type (book, course, video, article) and difficulty level, converts the output to HTML, and sends a formatted email.

## Tools Used
- **Orchestration:** n8n
- **LLM:** Google Gemini 1.5 Flash
- **Data:** HN Firebase API + HN search
- **Output:** Email via SMTP

## Flow
```
Form Trigger (topic + email)
  → Search HN Ask threads
  → Split out comment IDs
  → Fetch each comment
  → Aggregate all text
  → Gemini resource extraction + categorization
  → Convert Markdown → HTML
  → Send email
```

## Why I built this
The best learning recommendations I've ever seen come from HN threads. This surfaces them for any topic, automatically.
