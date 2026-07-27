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
| 5 | WhatsApp Agent | Draft-reply-approve-send on WhatsApp | ✅ Done |
| 6 | Multi-Agent Orchestration | All 3 channels unified into one orchestrator | 🔶 In Progress |
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


## Day 5 — Resolved
Full loop confirmed working: Dubai number sends message, Claude drafts a
friendly/professional reply, approval request sent to Homa's Canada
number, YES reply approves, final message delivered to Dubai number,
Google Sheet logs all six fields correctly.

Root causes found and fixed:
- n8n's native Twilio node had two separate issues sending via the shared
  sandbox number (error 21660 account mismatch, error 21211 double
  whatsapp prefix from the To Whatsapp toggle), replaced both Twilio
  send nodes with generic HTTP Request nodes calling Twilio's REST API
  directly with Basic Auth, which resolved it reliably.
- Anthropic node occasionally returns a thinking block before the reply
  text, fixed by using content.find(c => c.type equals text).text
  instead of content[0].text.
- HTTP Request nodes don't pass through input fields into their output
  (only the API response), so downstream nodes needing custom fields
  must reference the originating node by name (Get row(s), Webhook)
  rather than json directly.
- Get row(s) needs a Limit(1) node or better filtering, old pending
  rows accumulate in pending_approvals since status is never updated
  after send, which caused batch-processing errors. Add for Day 7.

## Day 5 — Post-test fixes
- Fixed: added a Limit(1, Keep Last) node between Get row(s) and If1,
  resolving the duplicate-send/duplicate-log issue caused by stale
  pending rows accumulating (previously listed as a Day 7 item).
- Fixed: added an instruction to the AI Draft system prompt telling
  Claude to output only the final reply text, no visible reasoning or
  self-corrections, which had been leaking into drafts occasionally.
- Still open for Day 7: explicitly update pending_approvals row status
  to approved/edited after send, rather than relying on Limit(1) alone.

## Day 6 — Progress Notes
Architecture: unified orchestrator built as a shared "Approval Hub" sub-workflow
using n8n's Execute Workflow Trigger + Wait node (Resume: On Form Submitted),
called from each channel workflow via Execute Workflow (Wait For Sub-Workflow
Completion enabled). This replaces the Day 5 approach of parsing WhatsApp
replies for approval, which only worked for a single channel and required
distinguishing sender vs approver by phone number.

Approval Hub flow: receives channel, sender, incoming_message, draft_text ->
sends a WhatsApp notification (via HTTP Request/Twilio, same as Day 5's fix)
containing a one-time form link -> Wait node pauses until the form is
submitted -> returns the submitted text to the calling workflow.

Completed and tested end to end:
- WhatsApp Agent v2 - Orchestrated: rebuilt to drop the old reply-parsing
  logic (If/Get row(s)/pending_approvals/Limit/If1/Edit Fields chain) in
  favor of a single call to the Approval Hub. Original Day 5 workflow left
  untouched as a backup.
- Email Agent v2 - Orchestrated: full rebuild from the Day 2 draft-only
  version. Replaced manual/batch trigger with a real Gmail Trigger, added
  a NO_REPLY_NEEDED branch (skips approval and logs as "skipped" when the
  existing system prompt determines no reply is warranted), wired the AI
  draft to the Approval Hub, and replaced "Create a draft" with an actual
  Gmail "Reply to a message" send.
- Shared Google Sheet log: added a Channel column so WhatsApp and Email
  both log to one unified sheet (Timestamp, From, Incoming Message,
  Drafted Reply, Approval Status, Final Sent Text, Channel).

Bugs found and fixed:
- Wait node's Form Description/Placeholder/Default Value fields do not
  reliably evaluate expressions (confirmed known n8n limitation, not a
  config error) - worked around by putting all dynamic context into the
  WhatsApp notification message instead, which renders correctly.
- Approval Hub's Wait node resume URL defaults to localhost, not the
  ngrok domain, since n8n's WEBHOOK_URL env var was never set to the
  tunnel address - approval links must be opened on the same Mac running
  n8n (copy from WhatsApp into Chrome) rather than tapped on phone.
  Fix for later: set WEBHOOK_URL to the ngrok domain.
- WhatsApp notifications fail silently (Twilio error 63016, outside the
  24-hour freeform messaging window) if Homa's number hasn't messaged the
  sandbox recently - needs a message sent to the sandbox to refresh the
  window before each testing session. Production fix would use an
  approved Message Template instead of freeform text.
- HTML entities (e.g. &#39; &gt;) from Gmail's snippet field show
  un-decoded in the WhatsApp notification text - cosmetic, not fixed yet.

Not done yet: LinkedIn Post Agent has not been wired to the Approval Hub.
Current LinkedIn workflow (chat trigger -> AI draft -> write file, no
approval step at all) still needs the same treatment as Email/WhatsApp.
Carrying this forward as an open item, not blocking Day 7.
