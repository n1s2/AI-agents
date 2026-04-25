# FLOOWBOX - AI Meeting Scheduler Agent

Meeting requests come in, get classified by type and urgency, and the requester gets 3 slot options plus a prep agenda — automatically.

## What it does

Request arrives via webhook. GPT-4o identifies meeting type (discovery, demo, followup), urgency, and generates 3 optimal slots within the requester's preferred times. Sends a slot-options email to the requester. Notifies the owner on Slack with what to prepare.

## Tools Used
- n8n, OpenAI GPT-4o, SMTP, Slack

## Setup
1. OpenAI API key, SMTP, Slack Bot + #calendar channel
