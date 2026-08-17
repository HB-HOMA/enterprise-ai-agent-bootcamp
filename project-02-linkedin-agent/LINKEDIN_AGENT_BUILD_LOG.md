# LinkedIn Agent — Build Log

Part of the `enterprise-ai-agent-bootcamp` portfolio (`project-02-linkedin-agent`). This agent generates, drafts, and (on approval) publishes LinkedIn posts on a schedule, built with n8n + Claude.

## Purpose

Unlike the Email and WhatsApp agents (which respond to inbound messages), this agent is proactive: it researches current news relevant to my career background, drafts an original LinkedIn post idea, and sends it to me for approval before publishing — 3x/week.

## Requirements

- AI generates post ideas grounded in my resume/career background and a fixed topic list: agentic AI, digital transformation, change management, product management, AI, management.
- Ideas are informed by live web research (market news, relevant companies) rather than static prompts — no dedicated newsletter API access, so this uses Claude's native web search tool instead of reading newsletter subscriptions.
- Must avoid repeating recently covered topics (checked against post history in the shared audit sheet).
- Human-in-the-loop approval before anything posts, reusing the same Approval Hub pattern as the Email and WhatsApp agents.
- Runs Monday/Wednesday/Friday, drafted and ready for review by 10:00 AM so posts can go out in the 10–11 AM window.
- Logged to the same shared Google Sheet audit trail (Channel = LinkedIn).

## Architecture

1. **Schedule Trigger** — Mon/Wed/Fri, 9:30 AM (buffer before the 10:00 AM review deadline).
2. **Google Docs node** — reads resume content as career/background context.
3. **Google Sheets node** — reads recent LinkedIn rows from the shared log, to avoid repeating topics.
4. **AI Agent (Claude + native Web Search tool)** — synthesizes resume, topic list, recent post history, and live web research into a full draft post.
5. **Approval Hub** (shared sub-workflow, reused from Email/WhatsApp) — sends the draft for review; I can approve as-is or edit before it goes out.
6. **LinkedIn node** — publishes the approved text (Post As: Person). LinkedIn's API defaults to comments enabled on new posts; no explicit toggle exists in n8n's LinkedIn node or was needed.
7. **Google Sheets append** — logs timestamp, topic, draft, approval status, final text, Channel = LinkedIn.

## Web research

Uses Tavily (free tier) as an HTTP Request Tool on the AI Agent, rather than a native Anthropic web search integration — n8n's Anthropic Chat Model node doesn't expose a web search toggle, and no bundled web search tool node was available. Tavily setup: POST to `api.tavily.com/search`, `query` param defined dynamically by the model, `api_key` and `max_results` sent as fixed body fields (Tavily requires the key in the body, not as a Bearer header, despite that also not raising an auth error — it just fails on the missing `query` field first).

## Setup notes / gotchas

- LinkedIn requires every developer app to be tied to a Company Page, even for apps that only post to a personal profile — created a placeholder page ("Homa's Content Agent") purely to satisfy this requirement.
- n8n's plain "LinkedIn OAuth2 API" credential type is correct for personal-profile posting (not "Community Management OAuth2," which requires LinkedIn's separate org-level app review and can't be scoped down to personal use).
- Required LinkedIn app products: "Share on LinkedIn" + "Sign In with LinkedIn using OpenID Connect."
- Credential settings: Organization Support **off**, Legacy **off** (Legacy uses deprecated scopes incompatible with the OpenID Connect product).
- OAuth tokens last 60 days with no refresh token on the free tier — credential needs manual reconnect periodically.
- Google OAuth "From list" file picker mode requires broader Drive scope than Sheets API alone grants — hits a 403. Workaround: use "By URL" mode instead, which only needs the specific-file scope already granted.
- Any node whose real output overwrites the item's `$json` (HTTP Request success response, LinkedIn "Create a post" response, etc.) will wipe out upstream fields referenced by bare `$json` in downstream nodes. Fix: reference the origin node explicitly by name — `$('Node Name').item.json.field` — instead of bare `$json.field`. Hit this bug three times across this build (Wait node form fields, LinkedIn Agent's sheet logging fields).

## Status — COMPLETE

- [x] LinkedIn OAuth credential connected
- [x] Workflow scaffolding (Schedule Trigger, Mon/Wed/Fri 9:30 AM)
- [x] Resume + post-history context wired into AI draft
- [x] Live web research via Tavily
- [x] Approval Hub + LinkedIn publish + sheet logging wired
- [x] End-to-end test — real post published and logged correctly

## Follow-ups (not yet done)

- Reusable pattern for running this agent for someone else: duplicate the workflow, swap LinkedIn credential, resume doc, and approval notification target.
- Sheet logging's `Topic` field currently just extracts hashtags from the post — fine for now, but could be made richer by having the AI Agent output a structured topic tag separately.

## Prompt update — Aug 17, 2026

Added two instructions to the AI Agent's drafting prompt after reviewing an external (ChatGPT) critique of a draft post:

1. Keep cited stats/reports clearly separated from the agent's own interpretation — don't blend "the data says X" and "which suggests Y" into one claim attributed to the source.
2. Repetition check now also compares recent posts for shared underlying thesis/story arc, not just overlapping topics or hashtags — catches cases where two posts make the same point with different numbers.
