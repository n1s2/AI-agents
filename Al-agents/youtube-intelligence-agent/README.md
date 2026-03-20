# FLOOWBOX - YouTube Channel Intelligence Agent

Built for content research and competitor analysis. Chat with any YouTube channel — ask about comment sentiment, top performing content, audience pain points, transcript summaries — the agent handles it.

## What it does

A chat-triggered AI Agent with a full tool suite for YouTube analysis. Tools cover channel details, video lists, comment scraping, transcript extraction, thumbnail vision analysis, and web search. Postgres stores the conversation memory so you can have multi-turn research sessions. Built for deep content intelligence work.

## Tools Used
- **Orchestration:** n8n
- **LLM:** OpenAI GPT-4o
- **Data:** YouTube Data API (Google Cloud)
- **Comment Scraping:** Apify
- **Memory:** Postgres Chat Memory
- **Vision:** OpenAI thumbnail analysis
- **Search:** Web search tool

## Agent Tools Available
| Tool | What it does |
|---|---|
| `get_channel_details` | Channel stats, subscriber count, description |
| `get_video_description` | Full video metadata |
| `get_list_of_videos` | All videos from a channel |
| `get_list_of_comments` | Comments via Apify scraper |
| `search` | Web search for additional context |
| `analyze_thumbnail` | Vision analysis of video thumbnails |
| `video_transcription` | Full transcript extraction |

## Example queries I've run with this
- "What are the top 3 pain points mentioned in comments on [channel]'s last 10 videos?"
- "Which video format gets the most engagement on this channel?"
- "Summarize the main themes in this video's comments"
- "Compare the thumbnail style of their top 5 videos"

## Why I built this

Content research for FLOOWBOX clients used to mean manually watching videos and reading comment sections. This agent does it in a conversation. Point it at a competitor's channel and ask anything.

## Setup
1. Google Cloud project → enable YouTube Data API → get API key
2. Apify account → API key for comment scraping
3. Postgres database for chat memory
4. OpenAI API key
