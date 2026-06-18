# Textpresso — design

The spec behind the tool. POC; not field-tested.

## Problem

Walking into daily standup having forgotten half of yesterday's work, plus prod issues and CI
regressions that should be raised. One trigger should reconstruct the picture.

## Decisions

| Decision     | Choice                                             | Why                                                                 |
| ------------ | -------------------------------------------------- | ------------------------------------------------------------------- |
| Form factor  | Claude Code skill, single trigger                  | All integrations (Slack/gh/Jira MCPs) already wired; no new backend |
| Action level | Read-only digest                                   | Safe; no outbound side effects. Drafting allowed, sending never     |
| CI/DX source | `gh` Actions API                                   | Fully scriptable today; proprietary DX portal deferred to v2        |
| Prod alerts  | Slack alert channels                               | Sentry intentionally skipped — alerts already land in Slack         |
| Scope        | Config file the user populates                     | Explicit, predictable, easy to tweak                                |
| Repo         | Public `kubaczerwinski77/textpresso`               | Drop-in install; safe because real config is gitignored             |
| Convention   | grill-me style (lean SKILL.md + referenced detail) | Low token cost, easy to extend                                      |

## Architecture

Plugin repo → two skills (`disable-model-invocation: true`, user-triggered):

- **`config`** — interactive setup; auto-derives via gh / Jira / Slack, writes the global config.
- **`brew`** — the briefing; thin orchestrator, per-source recipes + report layout in `references/`.

```
textpresso/
├── .claude-plugin/plugin.json
├── skills/
│   ├── config/SKILL.md          # interactive setup → ~/.config/textpresso/config.json
│   └── brew/
│       ├── SKILL.md             # lean orchestrator (gather → synthesize → render)
│       └── references/
│           ├── sources.md       # exact command/query per source, with caps
│           └── report-format.md # output layout + synthesis rules
├── textpresso.config.example.json
├── textpresso.config.json       # real config (gitignored; cwd-local)
├── README.md / LICENSE / .gitignore
```

Config resolution (`brew`): `$1` path → `./textpresso.config.json` → `~/.config/textpresso/config.json`.
The global path is what lets `brew` run from any repo; `config` is what writes it.

## Run sequence

1. Load config (abort with template if missing).
2. Resolve lookback (Mon → Friday; else yesterday).
3. Gather in parallel, bounded — only sources present in config.
4. Synthesize STANDUP TALKING POINTS (the value-add, not a raw dump).
5. Render report (`report-format.md`). Stop.
6. Stay interactive for follow-ups; drafting yes, sending no.

## Source → mechanism

| Section             | Source              | Mechanism                                 |
| ------------------- | ------------------- | ----------------------------------------- |
| Yesterday           | git + gh + Jira MCP | `git log`, `gh search prs`, Atlassian MCP |
| PR queue            | gh                  | `gh search prs`                           |
| Slack / Prod / News | Slack MCP           | bounded channel reads + mention search    |
| CI health           | gh Actions API      | `gh run list`                             |
| Talking points      | synthesis           | skill summarizes everything above         |

## Token discipline

Dominant cost is gathered data, not skill text. Mitigations: bounded reads (top-N per channel,
since-lookback only), concise synthesis, summaries over transcripts. If channel count grows,
v2 can fan out a subagent per source returning condensed summaries.

## Deferred (v2)

- Proprietary DX portal metrics
- Optional auto-actions (post standup, transition Jira) behind explicit confirmation
- Subagent fan-out for large channel sets
