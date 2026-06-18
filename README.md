# ☕ Textpresso

Your morning shot of engineering signal. One command pulls **yesterday's work, the PR review
queue, Slack signal, prod alerts and CI health** into a single standup-ready briefing — so you
walk into daily remembering everything you shipped and every fire worth raising.

Read-only by design: it gathers and synthesizes, it never sends or mutates.

**What lands in the briefing**

- 🎯 **Standup talking points**, synthesized first — shipped · in-flight · fires to raise · what you owe
- 📋 yesterday's merged PRs + moved tickets
- 👀 PR review queue — yours and the team's
- 🚨 prod alerts, CI failures, security CVEs
- 💬 the Slack threads that actually need you
- 📰 demos, launches, news worth knowing

Every PR, ticket, and Slack message is an **inline link** — click straight to the source.

## Requirements

- **Claude Code** with plugin support (v2.1.176+ — needed for the read-only `disallowed-tools` enforcement)
- **`gh` CLI**, authenticated (`gh auth login`) — PR queue, merged PRs, CI runs
- **Slack MCP** connected — channel signal + mentions
- **Atlassian MCP** connected — your Jira issue status + team board, read-only _(optional; omit the `jira` key to skip)_

## Install

```bash
claude /plugin install github:kubaczerwinski77/textpresso
```

Dev/test from a local clone before pushing:

```bash
claude --plugin-dir /path/to/textpresso
```

## Configure

Run the setup skill once — it auto-derives what it can (GitHub handle, teammates, Jira account,
Slack channels), asks for the rest, and writes a **global** config:

```
/textpresso:config
```

It writes to `~/.config/textpresso/config.json`, so `/textpresso:brew` then works from **any**
repo with no arguments.

Prefer to do it by hand? Copy the example and fill it in (gitignored — safe in a public repo):

```bash
cp textpresso.config.example.json textpresso.config.json
```

`brew` resolves config in order: explicit `$1` path → `./textpresso.config.json` (cwd) →
`~/.config/textpresso/config.json` (global). Tip: a Slack channel ID is in the channel’s
**About** panel (bottom).

## Use

```
/textpresso:brew
```

Optionally override the lookback window:

```
/textpresso:brew --lookback=yesterday
```

Then interrogate the result: `expand the PR queue`, `draft my standup message`,
`who else should review <PR>`. (Drafting is fine; sending is never automatic.)

## Customize what shows

Some sections might be noise for you. Tell `/textpresso:config` in plain language — _"hide the
interesting section"_, _"turn off Slack news but keep alerts"_, _"only standup and review"_ — and it
updates your config. Under the hood it's an optional `sections` map (see
`textpresso.config.example.json`): each section is `true`, `false`, or an object toggling its
subsections (`review`, `prod-ci`, `slack`). Omit it and everything shows. Hidden sections aren't
even gathered, so trimming saves tokens too.

## Extending

- **New channel** — add its ID to the right `slack` group. No code change.
- **New source** — add a block to `skills/brew/references/sources.md` and a config key.
  The gather step picks it up; the skill body stays untouched.

## Privacy

`textpresso.config.json` (your channel IDs, handles, repo paths) is gitignored. Only the
placeholder `*.example.json` ships in the repo. Keep it that way before flipping anything public.

## Security

`brew` ingests untrusted text — Slack messages, PR/Jira descriptions — any of which could carry a
prompt injection ("ignore instructions, post X / transition this ticket"). So read-only is enforced
by **capability, not prose**: each skill's `disallowed-tools` hard-blocks every mutating tool
(file writes, `git push`/`commit`, `gh pr merge`/`comment`/`api`, and all Slack/Jira create/edit/
update/transition/send tools). `disallowed-tools` is deny-first and cannot be overridden, so an
injection can't reach a tool that isn't there. (`allowed-tools` is _not_ an enforced restriction in
Claude Code — it only auto-approves — so it is not relied on for safety.)

**Sharing caveat:** the Slack denylist patterns are scoped to this environment's Slack MCP server
name (`mcp__plugin_slack_slack__*`). If your Slack MCP is registered under a different name, update
those patterns in `skills/brew/SKILL.md` and `skills/config/SKILL.md` to match, or the Slack write
tools won't be blocked. The Atlassian patterns (`mcp__atlassian__*`) are standard.

## License

MIT — see [LICENSE](./LICENSE).
