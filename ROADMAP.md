# 10-Day Enterprise AI Bootcamp Roadmap

Personal roadmap for building, documenting, and presenting an enterprise-grade
AI agent using n8n + Claude, as part of becoming an AI Solutions Architect / Consultant.

## Flagship Project: Personal Executive Assistant Agent
Pattern: Trigger -> AI Draft -> Human Approval -> Action -> Log
Channels: Email (Gmail) -> LinkedIn posts -> WhatsApp

| Day | Theme | Milestone Deliverable | Status |
|---|---|---|---|
| 1 | Foundations + Environment Setup | n8n, Claude API, GitHub repo all working | ✅ Done |
| 2 | Prompt Engineering + Email Agent v1 | Email -> Draft Reply -> Approval -> Send | ⏳ Next |
| 3 | Context Engineering + Voice/Tone | Agent writes in your voice using a context profile | |
| 4 | LinkedIn Post Agent | Topic -> Draft Post -> Approval -> Post (draft-only, no live posting) | ✅ Done |
| 5 | WhatsApp Agent | Draft-reply-approve-send on WhatsApp | 🔶 In Progress |
| 6 | Multi-Agent Orchestration | All 3 channels unified into one orchestrator | |
| 7 | Enterprise Hardening | Error handling, retries, logging, audit trail | |
| 8 | Security & Governance + Reskin | Security doc + adapted template for 2nd industry | |
| 9 | Documentation + Portfolio Build-out | Full repo docs, architecture diagrams | |
| 10 | Case Study + Presentation | Case study, LinkedIn article, resume bullets, demo | |

## Foundation Concepts Covered (Day 1)
- LLM = pattern-prediction engine, not a database of facts
- Tokens = pay-per-chunk of text (~4 characters each)
- Context window = the model's "whiteboard" size (fixed capacity)
- Temperature = creativity dial (low = predictable/safe, high = creative/varied)
- Hallucination = a confident but wrong guess, not a "bug"
- Automation vs. AI vs. AI Agent:
  - Automation = fixed rules, no thinking
  - AI = understands meaning, does one task, stops
  - AI Agent = decides, uses tools, acts across multiple steps toward a goal

## Environment / Tools
- Homebrew (package manager)
- Node.js v22 LTS via nvm (NOT the newest version — LTS is the enterprise standard)
- n8n v2.30.6 (workflow builder / "workshop")
- Anthropic Claude API (the agent's "brain")
- GitHub repo: enterprise-ai-agent-bootcamp

## How to Resume n8n on a New Session
1. Open Terminal
2. Run: nvm use 22   (make sure you're on Node 22, not system default)
3. Run: n8n start
4. Open browser to: http://localhost:5678 (use Chrome, not Safari)

## Day 7 — Planned addition
- Duplicate-content detection for LinkedIn Agent: check new draft
  topics/content against existing files in project-02-linkedin-agent/
  drafts/ before generating, to avoid repeating the same core message.
  Approach: inject past post content/summaries as context, add a rule
  instructing the model to avoid close duplication.

## Day 5 — Progress Notes
Architecture fully built in n8n:
- Trigger: Twilio WhatsApp Sandbox webhook (via ngrok tunnel, since local
  n8n's built-in --tunnel was removed in v2.0)
- AI Draft: Claude drafts a friendly/professional reply via system prompt
- Human Approval: draft and original sender sent to Homa's own WhatsApp for
  YES/edit decision, using an If node to distinguish approver replies from
  new incoming messages (requires 2 distinct WhatsApp numbers to test
  properly, solo testing with one number cannot simulate both roles)
- State: n8n Data table (pending_approvals) holds sender/draft/status
  between the initial draft and the approval reply
- Log: Google Sheets, one row per completed interaction (timestamp,
  sender, incoming message, draft, approval status, final sent text)

Blocked: final send-to-original-sender step failing on Twilio's side
(error 21660, mismatch between From number and account), confirmed
NOT an n8n config issue since Twilio's own Console test tool for the
sandbox also failed to load (Oops! Something went wrong) at the same
time. Likely a live Twilio platform issue, not something in our control.
Next session: retest once Twilio's sandbox tooling is responsive again.

Known follow-up items:
- Claude's Anthropic node output occasionally returns a thinking block
  as content[0] instead of the reply text, fixed by using
  content.find(c => c.type equals text).text instead of content[0].text
  wherever draft text is referenced.
- pending_approvals row status isn't auto-updated to approved/edited
  after send, fine for prototype, add for Day 7 hardening pass.
