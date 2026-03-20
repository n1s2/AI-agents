# FLOOWBOX - HN Trending Stories to Video Scripts

Content creation was eating too much time. This workflow pulls what's already trending and turns it into ready-to-record scripts.

## What it does

Fetches top stories from Hacker News, loops through them, generates structured video content scripts using OpenAI, creates matching images via an image generation API, and uploads all assets to Minio object storage.

## Tools Used
- **Orchestration:** n8n
- **Content Source:** Hacker News API
- **Script Generation:** OpenAI
- **Image Generation:** HTTP image API
- **Storage:** Minio (S3-compatible)

## Flow
```
Manual Trigger
  → Fetch top HN stories
  → Loop over each story
  → Generate video script (OpenAI)
  → Generate thumbnail image
  → Upload assets to Minio
```

## Why I built this
I wanted a week's worth of video scripts in one run. HN trending stories are pre-validated by the community — high engagement topics already.
