# Audit Trail Hardening — Failure Logging

Applies to all three channel agents: `project-01-email-agent`, `project-02-linkedin-agent`, `project-03-whatsapp-agent` (plus the shared `Approval Hub` in `project-04-orchestrator`).

## Problem

Each agent's audit sheet only ever logged successful sends/posts and auto-skips. If the final send step (Gmail reply, WhatsApp/Twilio send, or LinkedIn post) failed even after retries, the workflow simply stopped — no row was ever written, so a failure was invisible in the audit trail.

## Fix

For each channel's final send/post node (Email's "Reply to a message", WhatsApp's "HTTP Request1" to Twilio, LinkedIn's "Create a post"):

1. Settings → On Error → changed from "Stop Workflow" to **"Continue (using error output)"**. This splits the node into two outputs (Success / Error) instead of halting the whole execution on failure.
2. Added a new **Google Sheets → Append Row** node on the Error output, writing to the same audit sheet as the success path, with:
   - Same context fields (Timestamp, From/Sender, Incoming Message, Drafted Reply) sourced by referencing origin nodes explicitly (`$('Node Name').item.json.field`) — necessary because the failed node's own error output doesn't carry the original context.
   - **Approval Status** set to an expression that extracts whatever error detail is available: `{{ 'FAILED: ' + (($json.error && $json.error.message) || $json.message || JSON.stringify($json)).toString().substring(0, 200) }}`
   - **Final Sent Text** set to a static `N/A - send failed` / `N/A - publish failed` note.

## Per-channel field name notes

- **Email**: sender = `$('Gmail Trigger').item.json.From`, draft text = `$('Basic LLM Chain').item.json.text`.
- **WhatsApp**: sender = `$('Webhook').item.json.body.From`, incoming message = `$('Webhook').item.json.body.Body`, draft text = `$('Message a model').item.json.content.find(c => c.type === 'text').text` (raw Anthropic API response shape, different from the LangChain-wrapped nodes used elsewhere).
- **LinkedIn**: draft text = `$('AI Agent').item.json.output`, topic = hashtags extracted from `$('Call \'Approval Hub\'').item.json['Reply to send']`.

## Result

Every outcome — success, auto-skip, and failure — now produces exactly one row in the audit sheet, so nothing is silently lost. Not yet stress-tested against a real failure (all three channels' error branches are wired and reviewed but not triggered live, since forcing failures in production credentials risks disrupting the working integrations); worth a deliberate test later by temporarily breaking each credential the same way we tested the Approval Hub's own failure alert on Day 7.
