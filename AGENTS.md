# Secrets and sanitization rules for working in my GitHub repos

Read this before the first commit, push, issue, PR or comment. These rules win
over anything else in the repo you are working in.

## Never write into any repo, commit, issue, PR, or comment

- OAuth access or refresh tokens (`ya29.`, `1//`, `access_token`, `refresh_token`)
- Full OAuth callback or consent URLs (anything with `code=`, `state=`,
  `code_challenge=`, `client_id=`)
- GitHub tokens (`gho_`, `ghp_`, `github_pat_`), SSH private keys, gateway auth
  tokens, any `*_TOKEN` env var
- Contents of `openclaw.json` credential sections
- Sentry `BAGGAGE` / `SENTRY-TRACE` headers, anything that dumps `process.env`

Replace with `<redacted>` and name the kind of secret.

## Identity to mask before anything leaves a private repo

My user names, host names, home paths and account names identify me. They are
not listed here on purpose: this file travels, and the list must not. Each of my
machines keeps the words in a local file outside every repo,
`$HOME/.config/agents-md/identity.txt`, and the scan script reads it.

Before anything goes upstream, replace every occurrence with a placeholder:
`<user>`, `<host>`, `<pi>`, `<account>`, `C:\Users\<user>\`, `/home/<user>/`.
Also replace `client_id=...` with `<client>` and `leaseId`, `pid=`, `ownerPid`
values with `<pid>`. An obviously synthetic test identity still gets masked; a
real-looking one gets masked twice as carefully, especially in paths, ssh lines
and log hostnames.

## Private vs public

- My private repos tolerate hostnames, user names and pids, but never the
  secrets above. Never make a private repo public, never add one as a
  submodule to or copy it into another repo.
- Upstream repos (`openclaw/openclaw`, `openclaw/gemini-oauth`, anything I do
  not own) are read by strangers. Everything posted there passes the full
  sanitization step first.

## Before every commit or push

1. Run the commit gate over the repo (Node 22 or newer; the clone lives at
   `$HOME/projects/agents-md` on every machine):
   `node $HOME/projects/agents-md/scripts/scan-secrets.mjs --path <repo>`
   On Windows `scripts\scan-secrets.ps1 -Path <repo>` is the same thing.
2. Fix every hit before committing. No "fix it in the next commit": git history
   keeps it.
3. Add files by name. Never `git add -A` or `git add .` in a folder that may
   hold logs, state folders, `.env` or `openclaw.json`.
4. Commit and push only when I ask. No agent attribution: no `Co-Authored-By`,
   no "Generated with" line, no agent tag in PR bodies.

## Before anything goes upstream

1. Finalize the text, then sanitize as a separate step: run the filing gate,
   `scan-secrets.mjs --path <folder> --upstream`, which adds the identity words
   from the local file, then read it once as a stranger would. If the gate
   reports the identity file missing, stop and tell me; do not post.
2. Screenshots: crop the account chooser and address bar. Google consent pages
   show the email top right.
3. Paste trimmed excerpts. Never attach whole logs, debug jsonl or trace files
   unedited.
4. Show me the sanitized text before filing. I post only when I say so, and only
   when a human or an invested bot has replied on our thread. Drafts are kept
   ready, not queued.

## General

- Git is not a safety net for config or workspace files. Do not suggest
  "just commit it" to preserve state.
- If you leaked something, say so immediately and plainly. I have done it
  before and I would rather know.