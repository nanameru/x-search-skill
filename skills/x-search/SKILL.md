---
name: x-search
description: >-
  Search X (Twitter / 旧Twitter) for the latest posts, news, trends, or to
  collect post text and post URLs — OR look up a specific X post when the
  user pastes an x.com / twitter.com URL — by delegating to the local Hermes
  Agent one-shot mode. Use when the user says 「X検索」「Xで検索」
  「Xで最新情報」「Xで調べて」「ツイート検索」「Xのトレンド」「X search」
  「latest on X」「ディープリサーチ」「deep research」, asks for X/Twitter post URLs, or pastes any x.com /
  twitter.com link (post or profile). Hermes has Grok-backed X search built
  in — no X API key required (an X Premium account on this machine is required).
allowed-tools: Bash
---

# X Search (hermes -z)

Search X (Twitter) for live information by delegating to the local **Hermes
Agent** in one-shot mode. Hermes ships with Grok-backed X search, so **no X API
key is needed** — but the signed-in account must have **X Premium or higher**.

This skill works from any agent that can run shell commands (Claude Code, Codex,
Grok CLI). Install it with `npx skills add nanameru/x-search-skill --skill x-search`.

## The command

```
hermes -z "<prompt>"
```

`-z` / `--oneshot` runs Hermes once and prints **ONLY the final response text**
to stdout — no banner, no spinner, pipe-friendly. It is the `claude -p` /
`grok -p` equivalent.

Rules:
- **Always state the desired output shape in the prompt** (how many items, what
  format). Hermes returns plain text, so you control structure via the prompt.
- **🔴 ALWAYS transcribe the post text — every searched or fetched X post, no
  exceptions.** This includes filtered and numeric searches (min_faves /
  like-count, lang, date-range, etc.). Never return summary-only or "要点"-only
  results: every result must carry the post's actual text. Hermes must open and
  read each post and return the **full visible post text verbatim (文字起こし)**,
  plus author, handle, timestamp, post URL, and external URLs in the body.
  Display it per the copyright-limit rule below (short post = full quote; long
  post = 原文抜粋＋本文要約) — but the verbatim text itself must always be
  present. A result missing the post text is incomplete — re-run the search.
- **Full text via the `x_search` tool, not the browser.** In every prompt,
  explicitly instruct Hermes to retrieve post text using its `x_search` tool —
  do NOT rely on browser-based raw-text extraction (it is rate-limited and
  returns only quote/summary fragments, not verbatim text). If Hermes reports a
  browser limitation, re-run telling it to use `x_search` for the full text.
- **Relay post text within copyright limits.** In the final answer, include a
  "投稿本文" field for every result. If the post is short enough to quote in full
  within the current copyright policy limit, paste it verbatim. If it is longer,
  include a clearly labeled "原文抜粋" with no more than the allowed verbatim
  amount from that post, then add "本文要約" for the remainder. Do not present
  only the first line without saying it is an excerpt.
- **Do not bypass final-answer limits with Hermes output.** Even if Hermes
  prints a long post body in stdout, do not paste that long verbatim text
  through to the user. Transform it into the allowed excerpt plus a faithful
  summary, and keep the source URL prominent. If the user asks why, explain
  that Hermes can retrieve the text for grounding, but the assistant cannot
  redistribute long copyrighted/non-user-provided text verbatim.
- **Threads and quoted/reposted context:** If the relevant information is in a
  thread, ask for the full text of the relevant thread posts with each post's
  own URL. If a result depends on a quote post, include the quote post URL and
  full visible quote text when available.
- **🔗 ALWAYS return source links — mandatory.** Every X result MUST carry its
  source post URL. In search mode, each item keeps its own post URL. In link
  mode, echo back the original post URL AND every URL found in the post body
  (GitHub, articles, etc.). Never present X content without its source link:
  ask Hermes for URLs in the prompt, and verify they are present before
  replying. If a source URL is missing, re-run or say so explicitly.
- **NEVER add `--yolo`.** It disables tool-approval gates and will be blocked by
  the permission classifier. Plain `hermes -z` completes X-search tasks fine
  because search tools are read-only.
- It can take 30s–3min. Use a generous timeout; for long jobs run in background.

## Recommended prompt template

To search a topic and retrieve full post text:

```
hermes -z "X（旧Twitter）で「<TOPIC>」に関する最新情報を検索して、重要な関連投稿を<N>件取得してください。各投稿の本文は x_search ツールを使って（ブラウザ経由の抽出に頼らず）省略・要約せず全文取得し、番号付きリストで出力してください。各項目は『見出し / 投稿者名・@handle / 投稿日時（分かれば） / 投稿全文（Hermes側では省略・要約せず全文） / X投稿URL / 投稿内の外部リンク（あれば） / 補足メモ（必要な場合のみ短く）』の形式。スレッドの場合は関連するスレッド投稿の全文と各URLも含めてください。必ず各項目に実在するX投稿URLを含めてください。"
```

If the user gave no specific topic, pick a reasonable one (or use a broad
"最新の話題" search) and proceed — don't stall asking.

## Workflow

This skill has two triggers — **search mode** and **link mode**.

**A. Search mode** — user asks to search X / wants latest info or post URLs:
1. Build the prompt: topic + count + output format. Require Hermes to retrieve
   full post text for every result, not summary-only output.
2. Run `hermes -z "<prompt>"` (no `--yolo`, timeout ≥ 300s).

**B. Link mode** — user pastes an `x.com` / `twitter.com` URL (even with no
other text):
1. For a post URL, run:
   `hermes -z "次のX投稿を、x_search ツールを使って（ブラウザ経由の抽出に頼らず）開き、投稿本文を省略・要約せず全文で取得して。投稿者名・@handle・投稿日（分かれば）・投稿全文・投稿のURL（参照元）・本文中に含まれるURLを必ず併記して。スレッドなら関連するスレッド投稿も全文と各URL付きで出力して: <URL>"`
2. For a profile URL, retrieve that account's relevant recent posts with full
   visible post text — each item with its own post URL.

**Display the full text in link mode.** For a user-pasted link, show the entire
retrieved post body verbatim — **including long-form / X article posts**. The
user explicitly provided this URL, so do not truncate it to an excerpt. The
excerpt-only handling for long posts (step 3 / copyright rule) applies to bulk
search results, not to a single user-pasted link.

For a long-form / X article post, format the reply like this:
- A header line `**📄 <記事タイトル>（全文）**`, then bullets for
  投稿者（@handle）／日時, `🔗 投稿URL`, and 外部リンク.
- Then `投稿全文:` and the whole body, structured to mirror the source: the
  article's own section headings → bold; prose → blockquotes (`>`); code,
  JSON, diagrams, config → fenced code blocks; keep emoji, links and the
  author's wording verbatim.
- Close with a note on any X-UI cruft excluded (premium/publish prompt etc.),
  whether it was a thread or a single post with the reply count, then the
  one-line verification caveat.

For both modes:
3. Relay the result in the **Output format** below — include **投稿本文** for
   every item. For posts short enough to quote in full within the current
   copyright policy limit, paste the full returned body. For longer posts, label
   the verbatim part as **原文抜粋** and add **本文要約** for the rest. A result
   without retrieved post text or source links is incomplete; re-run or
   explicitly mark it incomplete. Do not paste long Hermes stdout verbatim just
   because Hermes already printed it.
4. **Add the verification caveat** (see below) — do not present results as
   confirmed-live without saying you can't verify from one-shot output.

## Output format

Present results to the user as a numbered list — one block per post, in this
exact shape (this is the preferred display):

```
**N. <短い見出し>**
- 投稿者: <表示名>（@handle）／ <投稿日 時刻>
- 投稿全文:
  > <投稿本文。短文ポストは全文そのまま。長文記事ポストは原文抜粋＋本文要約>
- 🔗 投稿URL: <X post URL>
- 外部リンク: <本文中のURL（あれば）／無ければ「なし」>
```

- One block per post. For a thread, one block per tweet in posting order.
- `投稿全文` is a blockquote — keep emoji, hashtags, and line breaks.
- `🔗 投稿URL` is mandatory on every block; never omit it.
- After the list, add the one-line verification caveat.

## Deep research mode

**Trigger:** the user says 「ディープリサーチ」「ディープリサーチを使用して」
「deep research」 (usually with a topic). Instead of a single query, this runs
an **iterative 5-round** X search that widens coverage each round.

The loop — 5 rounds, 5 distinct queries:
1. **Round 1** — run a normal search-mode `hermes -z` query for the topic.
2. **Analyze the gap** — compare what the results actually cover against the
   research goal. Name concrete gaps: unanswered sub-questions, missing angles,
   missing viewpoints / time periods / key accounts.
3. **Refine** — write a NEW query targeting the biggest gap. It must be a
   genuinely different angle, not a reword of the previous query.
4. Run the new query.
5. Repeat steps 2–4 until **5 rounds** are done. (Obey a different round count
   if the user explicitly asks for one.)

Rules:
- Every round obeys the 🔴 transcribe rule and the `x_search`-tool rule:
  verbatim post text + source URL for every result, in every round.
- De-duplicate — skip posts already found in earlier rounds.
- Keep all query strings; list them at the end so the research path is visible.
- If a round surfaces nothing new, say so; then continue, or stop early and
  explain why.

**Final output:**
1. Per-round results in the Output format above (grouped by round).
2. A **synthesis** — combined findings organized by sub-topic, what each round
   added, and the remaining open questions.
3. The list of all queries used, in order.
4. The verification caveat.

## Caveats — always tell the user

- One-shot mode (`-z`) hides tool-call logs, so you **cannot tell from the
  output whether Hermes ran a real live X search or generated plausible URLs.**
  Offer to verify by fetching one returned URL.
- `x.com/i/status/<id>` URLs are a generic redirect form; the ID being a real
  post is unverified until fetched.
- Requires X Premium on the signed-in account. If results look empty or an auth
  error appears, run `hermes status` or `hermes auth` to check the account.

## Beyond X search

`hermes -z` is a general single-turn agent call — usable for any one-shot task,
not just X search. Useful flags: `-m <MODEL>`, `-t <TOOLSETS>`,
`--skills <SKILLS>`, `--continue` (resume), `--worktree` (isolated git worktree).
