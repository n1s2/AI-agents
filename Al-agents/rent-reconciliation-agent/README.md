# FLOOWBOX - AI Rent Reconciliation Agent (Excel + OpenAI)

One of the more impactful workflows I've built for a client. A property manager was spending 4+ hours every month manually reconciling rent payments. This does it automatically.

## What it does

Watches a folder for incoming bank statement CSV files. When a new file lands, an AI agent reads it, cross-references each payment against a local Excel spreadsheet of tenant and property records, checks amounts and due dates while considering contract exceptions and tenant notes, then appends a structured alert report back to the spreadsheet for any discrepancies found.

## Tools Used
- **Orchestration:** n8n (self-hosted)
- **AI Agent:** OpenAI GPT-4o (with tool calling)
- **Data:** Local filesystem — CSV bank statements + XLSX tenant records (SheetJS)
- **Output:** Structured JSON alerts appended to Excel

## Flow
```
File Watcher (bank statement CSV drops)
  → Read CSV + parse rows
  → AI Agent with tools:
      - get_tenant_details (queries Excel)
      - get_property_details (queries Excel)
  → Structured Output Parser
  → Split alert items
  → Append to spreadsheet alerts sheet
```

## Why I built this
Manual reconciliation is pure overhead. The AI agent handles edge cases like partial payments, late payments within a grace period, and contract exceptions — things a simple script would miss.

**Note:** Designed for self-hosted n8n with local filesystem access.
