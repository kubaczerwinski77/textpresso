# ☕ Textpresso

Your morning shot of engineering signal. One command pulls **yesterday's work, the PR review
queue, Slack signal, prod alerts and CI health** into a single standup-ready briefing — so you
walk into daily remembering everything you shipped and every fire worth raising.

Read-only by design: it gathers and synthesizes, it never sends or mutates.

## Requirements

- **Claude Code** with plugin support
- **`gh` CLI**, authenticated (`gh auth login`) — PR queue, merged PRs, CI runs
- **Slack MCP** connected — channel signal + mentions
- **Atlassian MCP** connected — Jira transitions _(optional; omit the `jira` config key to skip)_

## Install

```bash
claude /plugin install github:kubaczerwinski77/textpresso
```

Dev/test from a local clone before pushing:

```bash
claude --plugin-dir ~/Text/textpresso
```

## Configure

Copy the example and fill in your scope. Your real config is gitignored — safe in a public repo.

```bash
cp textpresso.config.example.json textpresso.config.json
```

```jsonc
{
  "lookback": "last-workday", // Mon → scans Fri; else yesterday
  "github": {
    "me": "your-gh-handle",
    "team": ["teammate-one"], // teammate handles for the team PR queue
    "repos": ["/abs/path/to/repo"], // absolute paths — used for git log + CI
  },
  "jira": { "account": "your-id", "team": "your-team" },
  "slack": {
    "alerts": ["C0..."], // incident/alert channels → prod issues
    "team": ["C0..."], // team channels
    "news": ["C0..."], // content/news channels
  },
}
```

Tip: get a Slack channel ID from the channel’s **About** panel (bottom).

## Use

```
/textpresso:textpresso
```

Optionally pass a config path or lookback override:

```
/textpresso:textpresso ./textpresso.config.json --lookback=yesterday
```

Then interrogate the result: `expand the PR queue`, `draft my standup message`,
`who else should review <PR>`. (Drafting is fine; sending is never automatic.)

## Extending

- **New channel** — add its ID to the right `slack` group. No code change.
- **New source** — add a block to `skills/textpresso/references/sources.md` and a config key.
  The gather step picks it up; the skill body stays untouched.

## Privacy

`textpresso.config.json` (your channel IDs, handles, repo paths) is gitignored. Only the
placeholder `*.example.json` ships in the repo. Keep it that way before flipping anything public.

## License

MIT — see [LICENSE](./LICENSE).
