# AGENTS.md

Instructions for any AI agent (Claude Code, etc.) working in this repo.

## What this repo is

Keep the Receipt: an open-source (Apache-2.0) Chrome extension that pulls a user's order
invoices from retailers (Amazon US first) once a day, in the session they're already logged into,
renders each as a searchable PDF, and delivers it to outputs the user configures (a Downloads
folder, Paperless-ngx, a webhook, and more later). It runs locally, with no backend. Design:
`docs/design.md`.

**This repo is public, and it's meant for other people and agents to learn from.** Keep it
independent of any one person's setup: no personal data, no hostnames, tokens, paths or account
details from the maintainer's own machines, in code, docs, fixtures, commits or PR bodies. A
maintainer's test setup belongs in their own private notes.

## Git commits

- Prefix every agent-authored commit subject with `[claude] ` so agent commits
  are distinguishable from the maintainer's (Pat's) own at a glance in `git log --oneline`.
- Append a `Co-Authored-By` trailer naming both the model's display name and
  its exact model ID, so the specific model version is recoverable later even
  as display names change:
  ```
  Co-Authored-By: Claude Sonnet 5 (claude-sonnet-5) <noreply@anthropic.com>
  ```
  (Substitute the actual model running the session — check the session's own
  model ID rather than assuming this example.)

## Pull requests

- **This repo squash-merges PRs** — the PR description becomes the permanent
  commit message on `main`, not any individual commit's message. Write the PR
  body accordingly (if this repo does *not* squash-merge, delete this line and
  say what it does instead).
- **Every time additional commits are pushed to an already-open PR's branch,
  update the PR title/body before ending the turn** so it describes the full
  current diff, not just what was true when the PR was first opened.
- Fill out the PR template (if one exists) with a genuine step-by-step **test
  plan** a human can walk through manually — not just a restatement of
  lint/typecheck/build output. Check off what was actually verified this
  session; leave manual/live-only checks unchecked.
- Any comment an agent posts to a PR or issue (`gh pr comment`,
  `gh issue comment`, inline review comments) should start with a `[claude]`
  marker so it's identifiable as agent-authored.

## Tests

- **Add or update a test in the same PR as the code change** — it's part of
  "done," not a follow-up.
  - Bug fixes: add a test that fails without the fix, so the fix stays fixed.
  - New logic: add a unit/integration test alongside it.

## Secrets

Never print, log, or commit credentials, tokens, API keys, or similar
secrets. Check any content before writing it to GitHub (commit, comment, PR
body) for accidental inclusion.

## Test fixtures (retailer HTML)

Fixtures are real retailer pages, so they start out full of personal data. Before committing one,
scrub names, addresses, emails, phone numbers, order IDs, tracking numbers and card digits,
replacing each with a placeholder of the same shape. The PII check in CI must pass. Never weaken
that check to get a fixture through.

## Keeping this file current

If you learn something new working in this repo that a future agent would
want to know — a gotcha, a non-obvious convention, a workflow quirk — add it
here.
