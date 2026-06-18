---
name: config
description: Interactive setup and editor for textpresso. Auto-derives what it can (GitHub handle, teammates, Jira account, Slack channels), asks for the rest, and writes a global config so /textpresso:brew works from any repo. Also edits an existing config from natural-language commands — including hiding/showing report sections. Run after install, or any time to update.
disable-model-invocation: true
allowed-tools: Bash(gh api:*), Bash(gh search *), Bash(git rev-parse *), Bash(git * remote get-url *), Bash(mkdir *), Bash(cat *), Write, mcp__atlassian__atlassianUserInfo, mcp__atlassian__searchJiraIssuesUsingJql, mcp__atlassian__getVisibleJiraProjects, mcp__plugin_slack_slack__slack_search_channels, mcp__plugin_slack_slack__slack_search_public
disallowed-tools: Bash(rm:*), Bash(mv:*), Bash(git * push*), Bash(git * commit*), Bash(gh pr merge:*), Bash(gh pr close:*), Bash(gh pr comment:*), Bash(gh secret:*), Bash(gh workflow:*), mcp__atlassian__create*, mcp__atlassian__edit*, mcp__atlassian__update*, mcp__atlassian__transition*, mcp__atlassian__add*, mcp__plugin_slack_slack__slack_send*, mcp__plugin_slack_slack__slack_schedule*, mcp__plugin_slack_slack__slack_create*, mcp__plugin_slack_slack__slack_update*, mcp__plugin_slack_slack__slack_add_reaction
model: sonnet
effort: medium
---

# ☕ textpresso:config — setup

Build `$HOME/.config/textpresso/config.json` through a short Q&A. **Auto-derive everything
possible, show the user each value to confirm, and ask only for the subjective choices.**
Degrade gracefully: if a tool or MCP is missing, skip that auto-step and ask the user to type
the value instead. Never hard-fail.

## 0. Existing config?

If `$HOME/.config/textpresso/config.json` already exists, print it and ask what to do:
**overwrite** (full setup), **edit** (change a field or the layout), or **cancel**. Don't clobber silently.

**Natural-language edits.** On the edit path, accept plain-language commands and apply them to the
config, then write it back — e.g. "hide the interesting section", "turn off Slack news but keep
alerts", "only show standup and review", "add channel C0… to alerts", "look back 3 days". Map
layout requests to the `sections` schema in step 5. Show the change, confirm, write.

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

First capture two values used for inline links + mention detection:

- **`workspace`** — the subdomain in any Slack permalink (`https://<workspace>.slack.com/…`). Run
  one `slack_search_public` and read it off the returned permalink. Confirm.
- **`me`** — your Slack member ID (`slack_search_public` reports the current user's id). Capture it.

Then, for each bucket — **alerts / team / news** — help the user choose channels:

- Ask what to search (e.g. "alerts", "app-core"), call Slack MCP `search_channels`, show each
  result as `name — ID`, and let them pick which belong in this bucket. Repeat until the bucket
  is done, then move to the next.
- Store the channel **IDs** (`C0…`), never names.
- If the Slack MCP is absent, ask the user to paste IDs per bucket.

## 4. Lookback

Confirm the default `last-workday`, or take an override (`yesterday` | `N-days`).

## 5. Sections / layout (optional)

By default every section shows. Ask whether any are noise for this user, and write a `sections`
map (omit it entirely to keep the v0.1.0 default of everything-on). A section can be `true` (show),
`false` (hide), or an object toggling its subsections:

```jsonc
"sections": {
  "standup": true, "yesterday": true, "interesting": false,
  "review":  { "fresh": true, "team": true, "yours": false },
  "prod-ci": { "ci": true, "errors": true, "security": true },
  "slack":   { "alerts": true, "team": false, "news": false, "mentions": true }
}
```

Rules: absent section or `true`/`{}` = shown; `false` = hidden; object = shown with the listed
subsections off. Canonical keys live in `report-format.md` → Section visibility. Hidden sections
aren't gathered, so trimming also saves tokens.

## 6. Write

- `mkdir -p $HOME/.config/textpresso`
- Write the assembled JSON to `$HOME/.config/textpresso/config.json`. Include **only** the keys
  the user filled — a missing `jira` or `slack` key means `brew` skips that section.
- Echo the final config back and confirm success.
- Tell the user: **run `/textpresso:brew` from any repo now** — no arguments needed.

## Notes

- This config lives outside any repo. It's never committed — safe by construction.
- Re-run this skill any time to update scope or layout; it's idempotent (see step 0).
- `brew` is read-only and can't write config — all changes (including hiding sections) flow through this skill.
