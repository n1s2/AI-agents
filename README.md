{
  "name": "FLOOWBOX - Client Onboarding Email Sequence",
  "meta": {
    "instanceId": "navtej-floowbox-n8n-2025"
  },
  "nodes": [
    {
      "id": "node-001",
      "name": "New Client Trigger",
      "type": "n8n-nodes-base.webhook",
      "position": [240, 300],
      "parameters": {
        "path": "new-client",
        "httpMethod": "POST"
      },
      "typeVersion": 2
    },
    {
      "id": "node-002",
      "name": "Set Client Info",
      "type": "n8n-nodes-base.set",
      "position": [460, 300],
      "parameters": {
        "assignments": {
          "assignments": [
            {"id": "c1", "name": "client_name", "type": "string", "value": "={{ $json.name }}"},
            {"id": "c2", "name": "client_email", "type": "string", "value": "={{ $json.email }}"},
            {"id": "c3", "name": "service_type", "type": "string", "value": "={{ $json.service }}"},
            {"id": "c4", "name": "company", "type": "string", "value": "={{ $json.company }}"}
          ]
        }
      },
      "typeVersion": 3.4
    },
    {
      "id": "node-003",
      "name": "Generate Welcome Email",
      "type": "@n8n/n8n-nodes-langchain.chainLlm",
      "position": [680, 300],
      "parameters": {
        "promptType": "define",
        "text": "=Write a warm, professional welcome email for a new FLOOWBOX client.\n\nClient: {{ $json.client_name }}\nCompany: {{ $json.company }}\nService they signed up for: {{ $json.service_type }}\n\nThe email should:\n1. Welcome them personally\n2. Confirm what we'll be building for them\n3. Outline next steps (discovery call, requirements doc, timeline)\n4. Feel human, not templated\n\nSign off as Navtej, Founder - FLOOWBOX\nKeep it under 200 words."
      },
      "typeVersion": 1.5
    },
    {
      "id": "node-004",
      "name": "OpenAI GPT-4o",
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "position": [680, 460],
      "parameters": {
        "model": "gpt-4o",
        "options": {}
      },
      "typeVersion": 1
    },
    {
      "id": "node-005",
      "name": "Send Day 0 Welcome",
      "type": "n8n-nodes-base.emailSend",
      "position": [920, 300],
      "parameters": {
        "toEmail": "={{ $('Set Client Info').item.json.client_email }}",
        "fromEmail": "navtej@floowbox.com",
        "subject": "=Welcome to FLOOWBOX, {{ $('Set Client Info').item.json.client_name }}!",
        "html": "={{ $json.text }}",
        "options": {}
      },
      "typeVersion": 2.1
    },
    {
      "id": "node-006",
      "name": "Wait 1 Day",
      "type": "n8n-nodes-base.wait",
      "position": [1140, 300],
      "parameters": {
        "amount": 1,
        "unit": "days"
      },
      "typeVersion": 1.1
    },
    {
      "id": "node-007",
      "name": "Send Day 1 Checklist",
      "type": "n8n-nodes-base.emailSend",
      "position": [1360, 300],
      "parameters": {
        "toEmail": "={{ $('Set Client Info').item.json.client_email }}",
        "fromEmail": "navtej@floowbox.com",
        "subject": "=Getting started — what I need from you",
        "html": "=<p>Hi {{ $('Set Client Info').item.json.client_name }},</p><p>To kick off your {{ $('Set Client Info').item.json.service_type }} project, I need a few things from you:</p><ul><li>Access to your current tools/platforms</li><li>A 30-min discovery call (reply to book)</li><li>Any existing documentation or SOPs</li></ul><p>Once I have these, we can start building immediately.</p><p>— Navtej</p>",
        "options": {}
      },
      "typeVersion": 2.1
    },
    {
      "id": "node-008",
      "name": "Wait 3 Days",
      "type": "n8n-nodes-base.wait",
      "position": [1580, 300],
      "parameters": {
        "amount": 3,
        "unit": "days"
      },
      "typeVersion": 1.1
    },
    {
      "id": "node-009",
      "name": "Send Day 4 Check-in",
      "type": "n8n-nodes-base.emailSend",
      "position": [1800, 300],
      "parameters": {
        "toEmail": "={{ $('Set Client Info').item.json.client_email }}",
        "fromEmail": "navtej@floowbox.com",
        "subject": "Quick check-in",
        "html": "=<p>Hi {{ $('Set Client Info').item.json.client_name }},</p><p>Just checking in — did you get a chance to look at my previous email? Happy to jump on a quick call this week to get things moving.</p><p>— Navtej, FLOOWBOX</p>",
        "options": {}
      },
      "typeVersion": 2.1
    },
    {
      "id": "node-sticky-1",
      "name": "Sticky Note",
      "type": "n8n-nodes-base.stickyNote",
      "position": [220, 140],
      "parameters": {
        "content": "## FLOOWBOX - Client Onboarding Sequence\nTriggered when a new client signs up.\n\nSequence:\n- Day 0: Personalized welcome (GPT-4o written)\n- Day 1: Requirements checklist\n- Day 4: Check-in if no response\n\nEvery email is personalized using client name, company, and service type."
      },
      "typeVersion": 1
    }
  ],
  "connections": {
    "New Client Trigger": {"main": [[{"node": "Set Client Info", "type": "main", "index": 0}]]},
    "Set Client Info": {"main": [[{"node": "Generate Welcome Email", "type": "main", "index": 0}]]},
    "Generate Welcome Email": {"main": [[{"node": "Send Day 0 Welcome", "type": "main", "index": 0}]]},
    "OpenAI GPT-4o": {"ai_languageModel": [[{"node": "Generate Welcome Email", "type": "ai_languageModel", "index": 0}]]},
    "Send Day 0 Welcome": {"main": [[{"node": "Wait 1 Day", "type": "main", "index": 0}]]},
    "Wait 1 Day": {"main": [[{"node": "Send Day 1 Checklist", "type": "main", "index": 0}]]},
    "Send Day 1 Checklist": {"main": [[{"node": "Wait 3 Days", "type": "main", "index": 0}]]},
    "Wait 3 Days": {"main": [[{"node": "Send Day 4 Check-in", "type": "main", "index": 0}]]}
  },
  "active": false,
  "settings": {"executionOrder": "v1"}
}
