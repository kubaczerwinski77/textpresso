# Report format

A briefing you SCAN in 30 seconds and CLICK into anything. **Descriptive, linked, grouped by
status — never a raw dump.** Markdown renders in the Claude Code terminal, so use inline links,
**bold**, and divider lines.

## Linking rules — link everything you reference

Weave each link onto a noun phrase inside the sentence. **NEVER** a trailing `- link`, a `(link)`
tag, or a bare URL. If you genuinely can't build a link, state the thing plainly — don't fake one.

- **PR** → link the number + title: `[#9064 unify CI Node setup](<pr-url>)` (use `url` from gh).
- **PR review / comment** → if the signal is a specific review comment, link THAT comment URL, not
  just the PR.
- **Jira** → link the key: `[AC-3404](<jira-site>/browse/AC-3404)`. `<jira-site>` = the resource
  `url` from the Atlassian lookup (e.g. `https://livechatinc.atlassian.net`).
- **Slack** → link a topic phrase to the message permalink, constructed as:
  `https://<slack.workspace>.slack.com/archives/<channelId>/p<ts-with-the-dot-removed>`
  (ts `1781692968.786889` → `p1781692968786889`). Thread reply → append
  `?thread_ts=<parent_ts>&cid=<channelId>`. (`slack_search_public` also returns ready permalinks.)
- **People** → name them in prose; the link rides on what they said or did, not on their name.

## Layout

Separate every section with a divider and an emoji + title header. Lead each section with ONE
descriptive sentence, then tight items grouped by status — one line each.

```
──────────────────────────────────────────────
☕ TEXTPRESSO · Thursday 2026-06-18 · since Wed 06-17
──────────────────────────────────────────────
```

**🎯 STANDUP** — the lead. 2–3 sentences of plain talking points with links woven in, then:

- **Shipped** — merged PRs / done tickets, inline-linked.
- **In flight** — your open PRs + your in-progress tickets, inline-linked.
- **⚠️ Raise** — prod + CI fires, each linked to its alert message / Sentry issue / PR.
- **Blocked / ask** — decisions you owe or need, linked to the thread.

**📋 YESTERDAY** — one sentence ("Merged 3 PRs, moved [AC-3393] to Done."), then items inline-linked.

**👀 REVIEW QUEUE** — grouped:

- **Fresh on you** — PRs awaiting your review (author in parens), inline-linked.
- **Team — in review** — teammates' PRs / Jira issues In Review you could pick up.
- **Yours** — only if action is needed; else collapse to one line ("3 stale drafts — clean up").

**🚨 PROD / CI** — the fires, grouped: **CI** (merge queue / pipeline state), **Errors** (top alerts
by impact — `Nk events / M users`, each linked), **Security** (CVEs with due date + owner, linked).

**💬 SLACK** — only threads that need YOU. One line each, link on the topic phrase. If none, omit
the whole section (don't print "nothing").

**📰 INTERESTING** — events / launches / news, one line each, linked.

## Trimming rules

- Omit any empty section entirely. Never print "nothing here".
- No exhaustive dumps. Cap each list at ~5; if more, end with a linked `+N more`.
- There is **no** standalone full team-board section. From `jira-team`, surface only **In Review**
  items (review candidates) under REVIEW QUEUE, and fold standout In Progress work into STANDUP.
- Synthesis over listing: prefer "the team's pushing the CI overhaul ([AC-3404], [AC-3343])" to a
  ten-row table.
