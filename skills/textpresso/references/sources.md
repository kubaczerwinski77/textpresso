# Source recipes

One block per source. The gather step runs only the blocks whose config key exists.
**Every read is since the resolved lookback timestamp.** Keep outputs tight — these caps are
the difference between a snappy, cheap run and a token blowout.

> gh flags vary by version. Treat the commands below as starting points; if a flag is rejected,
> fall back to the raw search-query form, e.g. `gh search prs "author:@me is:merged merged:>=<date>"`.

## yesterday — what you did

- **Commits** — for each path in `github.repos`:
  `git -C <repo> log --author="<github.me>" --since="<lookback>" --no-merges --pretty=format:'%h %s'`
- **Merged PRs** — `gh search prs --author=@me --merged --json title,url,repository -L 20`
  (filter client-side to merged ≥ lookback)
- **Jira moved** — Atlassian MCP: issues where `assignee = jira.account` updated since lookback;
  report status transitions only.

## pr-queue — needs review

- **For you** — `gh search prs --review-requested=@me --state=open --json title,url,author,repository,updatedAt -L 30`
- **Team's open PRs** — for each handle in `github.team`:
  `gh search prs --author=<handle> --state=open --json title,url,reviewDecision,updatedAt -L 30`
- **Your PRs status** — `gh search prs --author=@me --state=open --json title,url,reviewDecision,updatedAt -L 30`
  Flag **stale**: no update in > 2 days.

## slack — signal only

**Cap: last 50 messages per channel OR since lookback, whichever is smaller. Summaries, not transcripts.**

- **Mentions** — Slack MCP search for the user since lookback (treat as high priority).
- **`slack.alerts[]`** — read each channel → incidents / errors / regressions → feed **⚠️ Raise**.
- **`slack.team[]`** — read each → high-signal only: long threads, decisions, questions aimed at the team.
- **`slack.news[]`** — read each → shared links, announcements, FYI-worthy items.

## ci — pipeline health

- For each path/repo in `github.repos`:
  `gh run list --repo <owner/repo> --created=">=<lookback>" --json name,conclusion,createdAt,updatedAt -L 50`
- Compute: **failure rate**, **slowest workflows**, **repeated failures (flaky)**.
- Notable degradations → **⚠️ Raise**.
