# claudeline-dex-horthy-dumb-zones

A fork of [fredrikaverpil/claudeline](https://github.com/fredrikaverpil/claudeline), a minimalistic Claude Code status line written in Go.

This fork adjusts the context window progress bar based on [Dex Horthy's](https://www.youtube.com/watch?v=rmvDxxNubIg&t=493s) "Dumb Zone" concept. You get earlier visual warnings so you can stay in the smart zone during long Claude Code sessions.

## What we changed (and why)

### Wider context bar (10 chars instead of 5)

The context window is the most useful thing to watch during a session. A wider bar makes it easier to read at a glance, especially when you're mid-task and only catching the status line in your peripheral vision.

![Comparison of stock vs fork status bars](docs/comparison.png)

### Earlier colour thresholds (the "Dumb Zone")

Stock claudeline stays green until 70%, then yellow, then red near compaction. By 70% you're already deep in what Dex Horthy calls the **Dumb Zone**. It's too late to course-correct.

The concept comes from [Dex Horthy](https://www.youtube.com/watch?v=rmvDxxNubIg&t=493s). Your context window has roughly 168,000 tokens, and around the 40% mark you start getting diminishing returns. The more of the window you've used, the worse the model performs.

If your context is packed with MCP JSON dumps, file searches, test output, and UUIDs, you're doing all your actual work in the zone where the model is least capable.

The fix is **intentional compaction**. Regularly compress your context before you hit that zone.

How it works:

1. Condense your working state into a markdown summary (specific files, line numbers, decisions made, the problem being solved).
2. Start a fresh context window.
3. The new session picks up from the summary and works in the smart zone instead of wading through stale noise.

Our thresholds are designed around this workflow:

| Range | Colour | Zone | Meaning |
|-------|--------|------|---------|
| 0-40% | Green | Smart zone | Full performance. Work freely. |
| 41-60% | Yellow | Dumb zone | Wrap up your current task or compact. |
| 61%+ | Red | Danger zone | Compact now or start fresh. You don't want to be here. |

The compaction warning (`⚠`) from upstream still triggers at 80% as a final alert.

### Everything else is stock

All other features are unchanged from upstream: subscription plan label, 5-hour and 7-day quota bars, git branch display, credential handling, and caching.

## Installation

Requires [Go](https://go.dev/dl/).

1. Clone and build:

```bash
git clone https://github.com/smshd/claudeline-dex-horthy-dumb-zones.git
cd claudeline-dex-horthy-dumb-zones
go build -o ~/.local/bin/claudeline .
```

2. Add to `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.local/bin/claudeline -git-branch"
  }
}
```

3. Restart Claude Code.

## Flags

| Flag | Default | Description |
|------|---------|-------------|
| `-debug` | `false` | Write warnings and errors to `/tmp/claudeline-debug.log` |
| `-git-branch` | `false` | Show git branch in the status line |
| `-git-branch-max-len` | `30` | Max display length for git branch |
| `-version` | `false` | Print version and exit |

## Troubleshooting

### Quota bars not showing (429 error)

If you see the model name and context bar but the 5-hour and 7-day quota bars are missing, the usage API is likely returning a 429 rate limit error.

**Check if this is the problem:**

Add the `-debug` flag to your settings command:

```json
"command": "~/.local/bin/claudeline -git-branch -debug"
```

Then check the log:

```bash
cat /tmp/claudeline-debug.log
```

If you see `usage: fetch usage API: unexpected status 429`, the fix is to refresh your OAuth token.

**How to fix it:**

```bash
claude logout
claude login
```

Then restart Claude Code.

**Note:** This logs you out of all active Claude Code sessions. If you have other terminals running, they will need to re-authenticate. Best to do this when you're at a natural stopping point.

This is a known issue with Anthropic's OAuth usage endpoint ([reference](https://github.com/anthropics/claude-code/issues/22876)). It affects all claudeline versions, not just this fork. The token itself gets rejected for the usage endpoint, and a fresh login resolves it.

Once the quota bars are working, you can remove the `-debug` flag from your settings.

## Keeping up with upstream

```bash
git fetch upstream
git merge upstream/main
go build -o ~/.local/bin/claudeline .
```

## Credits

- [claudeline](https://github.com/fredrikaverpil/claudeline) by [@fredrikaverpil](https://github.com/fredrikaverpil) for all the hard work: OAuth, usage API, caching, and Go architecture
- [Dex Horthy](https://www.youtube.com/watch?v=rmvDxxNubIg&t=493s) for the Dumb Zone concept that inspired our threshold changes

---

Made by [Smashed Avo](https://smashed-avo.com), a digital product studio based in Australia.
