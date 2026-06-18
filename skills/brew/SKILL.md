---
name: brew
description: Your morning engineering briefing. Pulls yesterday's work, the PR review queue, Slack signal, prod alerts and CI health into one standup-ready report. Read-only — gathers and synthesizes, never sends. Invoke explicitly each morning.
disable-model-invocation: true
argument-hint: "[config-path] [--lookback=last-workday|yesterday|N-days]"
allowed-tools: Bash(git *), Bash(gh *), Bash(jq *), Bash(cat *), Bash(date *), mcp__*
---

# ☕ Textpresso — morning brief

Read-only orchestration: **gather → synthesize standup talking points → render one report → stay interactive.** Never sends or mutates anything.

## 1. Load config

Resolve the config in this order, first hit wins:

1. explicit path passed as `$1`
2. `./textpresso.config.json` (current working dir)
3. `$HOME/.config/textpresso/config.json` (global default, written by `/textpresso:config`)

If none exists or the JSON is invalid, tell the user to run `/textpresso:config` and stop.
**Never invent channel IDs or handles** — only use what the config provides.

## 2. Resolve lookback

Default `last-workday`: on Monday, scan since last Friday 00:00; otherwise since yesterday 00:00.
Override via config `lookback` or the `--lookback` flag (`yesterday` | `N-days`).

## 3. Gather — in parallel, bounded

Read `references/sources.md` for the exact command/query per source. **Run only the sources
present in config** (a missing `slack`/`jira`/`ci` key = that section is skipped). Issue the
independent calls in ONE message so they run concurrently. Respect the caps in `sources.md`
(top-N per channel, since-lookback only) — bounded reads are what keep a run token-light.

Sources: `yesterday` (git + gh + jira) · `jira-team` (Atlassian MCP) · `pr-queue` (gh) · `slack` (alerts / team / news) · `ci` (gh actions).

## 4. Synthesize

Read every result, then write **STANDUP TALKING POINTS first** — this synthesis is the value,
not the raw dump. Bucket into: **Shipped · In flight · ⚠️ Raise (prod/CI) · Blocked/ask.**

## 5. Render

Follow `references/report-format.md`. Then **stop** — read-only contract, no outbound actions.

## 6. Stay interactive

Offer follow-ups: `expand <section>`, `draft my standup message`, `who else should review <PR>`.
Drafting text is fine; **sending** anything (Slack post, Jira transition, PR comment) is not.

## Extending

- **New channel** → add its ID to the right `slack` group in config. No skill edit.
- **New source** → add one block to `references/sources.md` + a top-level key in config. The
  gather step picks it up automatically. SKILL.md stays untouched.
