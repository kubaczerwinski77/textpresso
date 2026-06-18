# Report format

Render as Markdown in this order. Omit any section whose source is absent from config or
returned nothing. Keep bullets terse — this is a glance-and-go briefing.

```
☕ TEXTPRESSO — <Weekday> <YYYY-MM-DD>   (scanning since <lookback date>)

🎯 STANDUP TALKING POINTS
  • Shipped:    <merged PRs / done Jira>
  • In flight:  <open PRs + in-progress Jira>
  • ⚠️ Raise:   <prod alerts, flaky/slow CI>
  • Blocked/ask: <stale PRs awaiting review, threads needing you>

📋 YESTERDAY
  • <commit / merged PR / Jira transition lines>

👀 PR REVIEW QUEUE
  • For you:  <title — author — link>
  • Team:     <title — status — link>
  • Yours:    <title — reviewDecision — ⏳ stale? — link>

🧭 TEAM WORK (Jira)
  • <key — summary — assignee — status>

💬 SLACK — needs attention
  • Mentions: <who / where / one-line gist>
  • Team:     <high-signal threads only>

🚨 PROD / ALERTS
  • <incident / error / regression — channel — link>

⚙️ CI HEALTH
  • <failure rate · slowest workflow · flaky workflow>

📰 INTERESTING
  • <link / announcement — source>
```

Rules:

- **Talking points are synthesized**, not copied — they summarize the sections below them.
- Every item that is a fuck-up or risk must also appear under **⚠️ Raise** in the talking points.
- Prefer one line per item. Link when a URL exists. No raw transcripts.
