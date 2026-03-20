# FLOOWBOX - Ultimate JS-Rendered Site Scraper (Selenium + OpenAI)

Most n8n scraping workflows break on modern JavaScript-heavy sites. This one doesn't.

## What it does

Manages a complete Selenium WebDriver browser session lifecycle: creates a session, navigates to the target URL, waits for JS to render, extracts the full rendered HTML, cleans up and deletes the session. OpenAI then processes the raw HTML into structured, clean data. Handles retries and session cleanup on failure.

## Tools Used
- **Orchestration:** n8n
- **Browser Automation:** Selenium WebDriver (via HTTP)
- **Data Structuring:** OpenAI GPT
- **Logic:** Session management, conditional retry, clean deletion on all paths

## Flow
```
Trigger (URL input)
  → Create WebDriver session
  → Navigate to URL
  → Wait for render
  → Extract full HTML
  → Clean WebDriver output
  → OpenAI structured extraction
  → Delete session (success or failure path)
```

## Why I built this
A client needed data from a portal that required login and rendered everything via React. Standard HTTP request nodes returned blank pages. This browser session approach gets the fully rendered DOM every time.

## Requirements
- Selenium Grid or standalone WebDriver accessible via HTTP
- OpenAI API key
