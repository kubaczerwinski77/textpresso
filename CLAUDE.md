# CLAUDE.md — textpresso development guide

Guidance for AI agents (and humans) working on this repo. Read before changing anything.
The design rationale lives in [DESIGN.md](./DESIGN.md); this file is the working contract.

## What this is

A Claude Code plugin that produces a **read-only morning standup briefing**. One trigger
(`/textpresso:brew`) gathers yesterday's work, the PR review queue, Slack signal, prod / CI /
security, and news into one linked, scannable report. `/textpresso:config` writes the user's config.

## Core principles — do not violate

1. **Lightweight, low-token.** A glance-and-go tool, not a platform. The dominant cost is gathered
   data, not skill text. Keep `SKILL.md` lean; push detail to `references/`. Bound every read
   (caps, since-lookback, summaries — not transcripts). Do **not** add heavy machinery, background
   services, or always-on subagent fleets. Reach for subagents only when one source's volume
   demands it, and have them return condensed summaries.
2. **Read-only, always.** `brew` never sends or mutates — enforced by `disallowed-tools` (see
   Security). Any new source must be a read. Drafting text for the user is fine; sending is never
   automatic.
3. **Backward compatible.** v0.1.0 is published. Config changes are **additive only** with safe
   defaults — never rename or remove a key, never require a new one. An existing config must keep
   working and produce the same behavior it did before the change.
4. **Synthesis over dumping.** Group by status, lead with talking points, trim empty sections, link
   everything inline. Never a raw list where a sentence will do.

## Architecture

```
.claude-plugin/plugin.json      # manifest (name, version — bump on release)
skills/
  config/SKILL.md               # interactive setup → ~/.config/textpresso/config.json
  brew/
    SKILL.md                    # lean orchestrator: load config → gather → synthesize → render
    references/
      sources.md                # per-source gather recipes + caps (loaded at runtime)
      report-format.md          # output layout, linking + trimming rules
textpresso.config.example.json  # schema template (committed)
textpresso.config.json          # real config (gitignored)
```

- **Config resolution (`brew`):** `$1` path → `./textpresso.config.json` → `~/.config/textpresso/config.json`.
- **grill-me convention:** keep `SKILL.md` short; recipes and format live in `references/` so the
  body is cheap to load and easy to extend.

## Config schema

`lookback` · `github{me,team,repos}` · `jira{account,project,team}` · `slack{workspace,me,alerts,team,news}`.
A missing top-level source key (`jira`, `slack`, `ci`) means that section is skipped. Keep
`textpresso.config.example.json` in lockstep with any schema addition, and update the `config` skill
so setup writes the new key.

## Report format

See `skills/brew/references/report-format.md`. Descriptive lead sentence per section, grouped by
status, inline links (PR `url`, Jira `…/browse/KEY`, Slack message permalinks via `search_public`),
no `---` dividers, omit empty sections.

## Security

Read-only is enforced by **`disallowed-tools`** (deny-first hard block), **not** `allowed-tools`
(Claude Code does not enforce that — it only auto-approves). If a skill gains a new capability,
keep every mutating variant in the denylist. The Slack denylist is scoped to the
`mcp__plugin_slack_slack__*` server name — env-specific; flag it for sharers.

## Model & cost

Both skills pin `model: sonnet`, `effort: medium` — cheap, and overrides the user's session model.
Don't pin Opus; the work is tool-calling plus light synthesis.

## Dev workflow

- **Test locally:** `claude --plugin-dir /path/to/textpresso`, then `/textpresso:brew`. After edits,
  `/reload-plugins` or start a fresh session.
- **Add a source:** add a block to `sources.md`, add an additive config key, let the gather step
  pick it up. Usually no `SKILL.md` change.
- **Release:** bump `version` in `.claude-plugin/plugin.json`, then `git tag vX.Y.Z` + push +
  `gh release create`. `/plugin install` users track releases, not HEAD.
- **Commits:** conventional commits (`feat:` `fix:` `docs:` `security:` `style:`). No Jira keys
  (personal repo).

## Active direction

**Per-user section visibility** — some users want fewer sections; the full set is noise for them.
Plan: config-driven section control, default = everything visible so existing v0.1.0 configs are
untouched. Additive and backward-compatible (principle #3). Design the schema before implementing.
