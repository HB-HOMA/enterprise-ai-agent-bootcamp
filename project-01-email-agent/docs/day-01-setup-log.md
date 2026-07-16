# Day 1 Setup Log — Environment & First AI Workflow

## Goal
Get a fully working local AI agent environment: n8n + Claude API + GitHub,
and run one successful end-to-end AI call.

## What I Installed (in order)
1. Homebrew — package manager for Mac
2. Node.js v26 (default) — later found to be too new/unstable for n8n
3. nvm (Node Version Manager) — to manage multiple Node versions
4. Node.js v22 (LTS) via nvm — set as default
5. n8n (v2.30.6) via npm install -g n8n
6. Claude API key via console.anthropic.com (Individual account, $5 credit)

## Problem I Hit + How I Solved It
**Problem:** `npm install n8n -g` failed while compiling a dependency
(isolated-vm) with a C++ error: "'memory' file not found"

**Root cause:** Node.js v26 was too new — no pre-built binary existed for it,
so npm tried building from source and failed on missing Apple developer headers.

**Fix:** Installed nvm, switched to Node.js v22 (the stable/LTS version),
reinstalled n8n — worked immediately, no errors.

**Lesson learned:** Always use LTS (Long Term Support) software versions for
client/professional work, not the newest release. Newest != most stable.

## n8n Setup Steps
1. Installed n8n globally via npm
2. Ran `n8n start`, opened http://localhost:5678 in Chrome (Safari had a
   secure-cookie error with local HTTP — Chrome avoided this)
3. Created local owner account
4. Added Anthropic API credential (Settings > Credentials > Add > Anthropic)
   — confirmed "Connection tested successfully"

## First Working Workflow
- Nodes used: "When chat message received" (trigger) -> "Basic LLM Chain"
  -> "Anthropic Chat Model" (Claude Sonnet 4.6)
- Prompt tested: "Explain what an AI agent is, in one sentence, to a business executive."
- Result: Successful run, 67 tokens, 2.5 seconds
- Output: "An AI agent is software that autonomously pursues a goal on your
  behalf — making decisions, taking actions, and adapting along the way —
  without needing step-by-step human instruction."

### Key lesson: "Anthropic Chat Model" alone has no prompt box —
it's an engine that must be attached to a parent node like "Basic LLM Chain"
which is where the actual prompt/instructions go.

## GitHub Setup
1. Created public repo: enterprise-ai-agent-bootcamp (with README + .gitignore)
2. Cloned to ~/Desktop/enterprise-ai-agent-bootcamp
3. Created project folder structure (workflows, docs, prompts, screenshots, shared)
4. Committed and pushed using a GitHub Personal Access Token (classic, repo scope)
   — GitHub no longer accepts account passwords for git push, token required instead
5. Added .DS_Store to .gitignore to keep the repo clean of Mac system files

## Commands to Resume Work Next Session

## Security Incident + Git Authentication Troubleshooting

### What happened
While pasting terminal output back into chat for documentation purposes, a
GitHub Personal Access Token was accidentally exposed in plain text (typed
directly into Terminal instead of at the hidden password prompt).

### Response (correct incident-response pattern)
1. Immediately revoked the exposed token at github.com/settings/tokens
2. Generated a new token (repo scope only, 90-day expiration)
3. Verified the new token worked using a direct API test:
   `curl -u USERNAME:TOKEN https://api.github.com/user`
   (returned valid account JSON = token confirmed working)

### Problem: git push kept failing even with a valid token
**Symptom:** `remote: Invalid username or token` even though curl confirmed
the token was valid.

**Root cause:** macOS Keychain had cached the OLD (deleted) token from a
previous login, and kept auto-supplying it to git push instead of prompting
fresh.

**Fix:**
This clears the cached credential. After that, `git push` correctly prompted
fresh for username + token, and authentication succeeded.

### Lessons learned
1. Never type a secret (token/password) directly as a terminal command —
   only paste it at a hidden password-style prompt. If it prints visibly on
   screen, that's a signal something is wrong.
2. When credentials "should work" but keep failing, suspect a CACHED old
   credential before assuming the new one is broken. Test the new credential
   in isolation first (e.g. via curl) to rule that out.
3. Incident response pattern for any exposed secret: revoke immediately,
   issue a new one, verify the new one works, THEN retry the original task.
   Don't skip the "revoke" step even for low-risk tokens.
