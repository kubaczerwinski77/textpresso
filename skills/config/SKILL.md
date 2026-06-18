---
name: config
description: Interactive setup for textpresso. Auto-derives what it can (GitHub handle, teammates, Jira account, Slack channels), asks for the rest, and writes a global config so /textpresso:brew works from any repo. Run once after install, or again to update.
disable-model-invocation: true
allowed-tools: Bash(gh *), Bash(git *), Bash(mkdir *), Bash(cat *), Write, mcp__*
---

# ☕ textpresso:config — setup

Build `$HOME/.config/textpresso/config.json` through a short Q&A. **Auto-derive everything
possible, show the user each value to confirm, and ask only for the subjective choices.**
Degrade gracefully: if a tool or MCP is missing, skip that auto-step and ask the user to type
the value instead. Never hard-fail.

## 0. Existing config?

If `$HOME/.config/textpresso/config.json` already exists, print it and ask: **overwrite**,
**edit one section**, or **cancel**. Don't clobber silently.

## 1. GitHub (auto)

- **Handle** — `gh api user --jq .login`. Confirm with the user.
- **Teammates** — ask for the org + team slug (e.g. `livechat` / `app-core-web-desktop`), then
  `gh api orgs/<org>/teams/<slug>/members --jq '.[].login' | sort`. Drop the user's own handle
  (it's already `github.me`). Show the list; let them add/remove. No GH team? Ask for handles directly.
- **Repos** — default to this repo's root (`git rev-parse --show-toplevel`). Ask for any other
  absolute paths to watch. Store as an array (v2 will lean on this for multi-repo).

## 2. Jira (auto, optional)

- **Account** — Atlassian MCP `atlassianUserInfo` → `account_id`. Confirm.
- **Project** — ask for the key (e.g. `AC`).
- **Team** — ask for the team name and resolve it to the value used by the Team custom field;
  if unsure, ask the user to paste the team UUID. (LiveChat's Team field is `customfield_10500` —
  other orgs differ; note this to the user.)
- If the Atlassian MCP is absent, ask whether to skip Jira entirely (omit the `jira` key).

## 3. Slack (interactive)

For each bucket — **alerts / team / news** — help the user choose channels:

- Ask what to search (e.g. "alerts", "app-core"), call Slack MCP `search_channels`, show each
  result as `name — ID`, and let them pick which belong in this bucket. Repeat until the bucket
  is done, then move to the next.
- Store the channel **IDs** (`C0…`), never names.
- If the Slack MCP is absent, ask the user to paste IDs per bucket.

## 4. Lookback

Confirm the default `last-workday`, or take an override (`yesterday` | `N-days`).

## 5. Write

- `mkdir -p $HOME/.config/textpresso`
- Write the assembled JSON to `$HOME/.config/textpresso/config.json`. Include **only** the keys
  the user filled — a missing `jira` or `slack` key means `brew` skips that section.
- Echo the final config back and confirm success.
- Tell the user: **run `/textpresso:brew` from any repo now** — no arguments needed.

## Notes

- This config lives outside any repo. It's never committed — safe by construction.
- Re-run this skill any time to update scope; it's idempotent (see step 0).
