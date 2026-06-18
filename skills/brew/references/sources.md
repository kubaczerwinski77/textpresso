# Source recipes

One block per source; the gather step runs only the blocks whose config key exists. **Every read
is since the resolved lookback.** Keep outputs tight AND capture the data needed for inline links
(PR `url`, Slack `channelId` + `ts`, Jira keys). The caps below keep a run cheap.

> gh flags vary by version. If a flag is rejected, fall back to the raw query form, e.g.
> `gh search prs "author:@me is:merged merged:>=<date>"`.

## Identifiers — resolve once, reuse for links

- **Jira site** — Atlassian `getAccessibleAtlassianResources` → resource `url`
  (e.g. `https://livechatinc.atlassian.net`). Use for `…/browse/<KEY>` links and as the cloudId source.
- **Slack** — prefer permalinks straight from `slack_search_public` (it returns ready links,
  including the `?thread_ts=…` thread form). `slack.workspace` is only the **fallback** for
  hand-building permalinks in **private** channels (search_public can't see them).
  `slack.me` (your member ID) finds mentions.

## yesterday — what you did

- **Commits** — per repo in `github.repos`:
  `git -C <repo> log --author="<github.me>" --since="<lookback>" --no-merges --pretty=format:'%h %s'`
- **Merged PRs** — `gh search prs --author=@me --merged --json number,title,url,repository -L 20`
  (filter to merged ≥ lookback). Keep `url`.
- **Jira moved** — `searchJiraIssuesUsingJql`, fields `["summary","status"]`, maxResults 20:
  `assignee = "<jira.account>" AND status CHANGED AFTER "<lookback>" ORDER BY updated DESC`.

## pr-queue — needs review

`reviewDecision` is **NOT** a valid field on `gh search prs` — do not request it. Use:

- **Fresh on you** — `gh search prs --review-requested=@me --state=open --json number,title,url,author,repository,updatedAt,isDraft -L 30`
- **Team open** — per handle in `github.team`:
  `gh search prs --author=<handle> --state=open --json number,title,url,repository,updatedAt,isDraft -L 30`
- **Yours** — `gh search prs --author=@me --state=open --json number,title,url,repository,updatedAt,isDraft -L 30`.
  Flag **stale** (no update > 2 days) and **draft**. Need a review decision on one specific PR?
  `gh pr view <url> --json reviewDecision` (single call, only when it matters).

## jira-team — what the team's working on (trimmed)

fields `["summary","status","assignee"]`, maxResults 20 — **never** pull descriptions or comments
(that's what blew the token cap). Two narrow queries:

- **In Review** (the useful slice):
  `project = <jira.project> AND cf[10500] = "<jira.team>" AND status = "In Review" ORDER BY updated DESC`
- **In Progress** (context only):
  `project = <jira.project> AND cf[10500] = "<jira.team>" AND statusCategory = "In Progress" ORDER BY updated DESC`

Per report-format: In Review → REVIEW QUEUE; standout In Progress → STANDUP. No full dump.

## slack — signal only

Cap: ~50 msgs/channel or since lookback, whichever is smaller. Summaries, not transcripts.

**Get permalinks from `slack_search_public`** — it returns ready-made links (including the
`?thread_ts=…&cid=…` thread form), so don't hand-build them for public channels.

- **Public channels** (alerts / news + any public team channel) — discover per channel with
  `slack_search_public`: `in:<#CHANNELID> after:<lookback-date>`, `sort=timestamp`. Each result
  carries its own `Permalink:` — use it directly.
- **Private channels** (🔒, e.g. `C08CJACGHH9` / `C08F2QH4PQD`) — `search_public` can't see them.
  Read with `slack_read_channel`, then construct the permalink from `channelId` + `ts` +
  `slack.workspace` (see Identifiers). Don't use `search_public_and_private` — it needs user consent.
- **Mentions** — `slack_search_public` with `to:me` plus your member id `<slack.me>`, after lookback.

Per bucket: **alerts[]** → incidents / errors / regressions (capture any Sentry/PR URL in the text)
→ **⚠️ Raise**; **team[]** → decisions, questions aimed at the team, threads needing you;
**news[]** → shared links, announcements, events.

## ci — pipeline health

Only if a `ci` key is present in config.

- Per repo, derive `owner/repo` from `git -C <repo> remote get-url origin`, then
  `gh run list --repo <owner/repo> --created=">=<lookback>" --json name,conclusion,createdAt,updatedAt -L 50`.
- Compute failure rate, slowest workflows, repeated failures (flaky). Degradations → **⚠️ Raise**.
