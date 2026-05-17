# x-search — Agent Skill

X (Twitter) search & full-text transcription for AI coding agents, powered by
the local **Hermes Agent** (`hermes -z`). No X API key required — it uses
Hermes' built-in Grok-backed X search. An **X Premium** account signed into
Hermes is required.

## Install

```bash
npx skills add nanameru/x-search-skill --skill x-search
```

### Claude Code

```bash
npx skills add nanameru/x-search-skill -g -a claude-code --skill x-search -y --copy
```

### Codex

```bash
npx skills add nanameru/x-search-skill -g -a codex --skill x-search -y --copy
```

### Both

```bash
npx skills add nanameru/x-search-skill -g -a claude-code -a codex --skill x-search -y --copy
```

Verify:

```bash
npx skills list -g
```

## What it does

- **「Xで検索して」** — searches X and transcribes every result post verbatim,
  each with its source URL.
- **Paste an x.com / twitter.com link** — fetches that post in full, including
  long-form X article posts.
- Filtered / numeric searches (`min_faves`, `lang`, date range) are supported,
  and every result still carries the verbatim post text.

## Requirements

- The Hermes Agent CLI installed and on `PATH` (`hermes`).
- An **X Premium (or higher)** account signed into Hermes.

## Notes

`hermes -z` (one-shot) hides tool logs, so live-search retrieval cannot be
verified from its output alone. Treat reported like/repost counts as
approximate, and verify specific posts by opening their URL when accuracy
matters.
