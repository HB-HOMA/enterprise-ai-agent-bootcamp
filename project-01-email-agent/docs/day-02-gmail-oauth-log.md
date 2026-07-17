# Day 2 — Gmail OAuth Integration Log

## Goal
Connect Gmail to n8n using OAuth2 so the email agent can read real inbox
messages and (later) send approved draft replies.

## Why OAuth (not just an API key)
Unlike Claude's API key (a simple secret string), Gmail requires OAuth2 —
a permission-based login flow. This is standard for any integration that
acts on a real user's personal/business account (Gmail, Outlook, Salesforce,
etc.), not just a "give me a key" system like most AI model APIs.

## Steps Completed
1. Created a Google Cloud project: n8n-bootcamp (had to enable 2-Step
   Verification on the Google account first — Google now requires MFA
   for Cloud Console access)
2. Enabled the Gmail API for the project
3. Configured the OAuth consent screen (Google Auth Platform):
   - App type: External
   - Added self as a Test User (required while app is unverified)
4. Created an OAuth 2.0 Client (Web application type) with redirect URI:
   http://localhost:5678/rest/oauth2-credential/callback
5. Copied Client ID + Client Secret into n8n's Gmail OAuth2 credential form
6. Completed Google sign-in flow inside n8n, saw the "Google hasn't
   verified this app" warning (expected/safe for a personal project
   I created myself) and clicked through
7. On the permission screen, granted ONLY the scopes actually needed
   (principle of least privilege):
   - "Read, compose, and send emails from your Gmail account"
   - "See and edit your email labels"
   Explicitly did NOT grant the broader "read, compose, send, and
   permanently delete all email" scope, since deletion isn't needed.

## Verification
Ran the "Get many messages" Gmail node in n8n -> successfully retrieved
50 real messages from the live inbox, confirming the OAuth connection
works end-to-end.

## Key Lesson
Scoped permissions matter. When connecting any real business account
(Gmail, CRM, ERP, etc.) for a client, always review the exact permission
list before approving, and grant only what the specific task requires —
never "select all" by default.
