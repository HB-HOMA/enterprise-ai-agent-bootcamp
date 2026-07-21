# Day 4 — LinkedIn Agent + File Writing Fix

## Known n8n bug: "The file is not writable"
Newer n8n versions (2.x) block ALL file writes by default -- even to
universally writable locations like /tmp -- unless explicitly told
which folders are allowed, via an environment variable.

## The fix (must be set BEFORE starting n8n, every session)
export N8N_RESTRICT_FILE_ACCESS_TO=/Users/homab/Desktop/enterprise-ai-agent-bootcamp/
nvm use 22
n8n start

## Updated startup routine going forward
1. Open Terminal
2. export N8N_RESTRICT_FILE_ACCESS_TO=/Users/homab/Desktop/enterprise-ai-agent-bootcamp/
3. nvm use 22
4. n8n start
5. Open http://localhost:5678 in Chrome

## Verified working
Confirmed a real .txt file (682 bytes) successfully written to
project-02-linkedin-agent/drafts/ after applying the fix. Full pipeline
(topic -> Claude draft -> Convert to File -> Write File) works end-to-end.

## Lesson learned
When an error persists across completely different file paths/locations
(project folder AND /tmp), the problem is not the path -- it's the
tool's own configuration. Searching for the exact error message led
directly to a known, documented n8n issue with a clear fix.
