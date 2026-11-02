---
name: english-review
version: 1.0.0
description: Review drill: pick a few of the user's past English mistakes from the english-practice MCP server, show only the original (wrong) sentences, let the user rewrite them, then grade each rewrite against the stored correction. Invoke with /english-tutor-plugin:english-review [error_type] [count].
argument-hint: [error_type] [count, default 3]
allowed-tools:
  - mcp__english-practice__list_error_types
  - mcp__english-practice__search_sentences
---

# english-review

A two-turn drill. Turn 1 shows old wrong sentences; turn 2 grades the user's rewrites.

## Turn 1: present the drill

1. **Parse the argument.** Optional `error_type` (a category such as `tense` or `preposition`) and optional `count` (1–10, default 3). If an error type is given, call `list_error_types` first and map the user's wording to an existing name (e.g. "verb tense" → `tense`); if nothing matches, say so in one line, list the available names, and stop.

2. **Call `search_sentences`** with `error_type` (if any) and `limit: 30`. Drop sentences whose `errors` list is empty. If fewer than one remains, reply with one line in 繁體中文 saying there is nothing to review (for that type), and stop.

3. **Pick `count` sentences**, preferring variety: different error types and different days. Prefer older sentences over ones saved today.

4. **Reply** in 繁體中文 in exactly this layout. Show ONLY the original sentences. Do not reveal corrections, error types, snippets or explanations yet.

```
複習練習（<count> 題<，錯誤類型：error_type if given>）
請把每一句改寫成正確的英文，然後回覆我：

1. <original_text>（id <id>）
2. <original_text>（id <id>）
3. <original_text>（id <id>）
```

## Turn 2: grade the rewrites

When the user replies with their rewrites (in the same conversation):

5. **Compare each rewrite** with the stored `corrected_text` and `errors` of that sentence. A rewrite passes if every stored error is fixed, even if the wording differs from `corrected_text`. Do not call `search_sentences` again; use the data from turn 1.

6. **Reply** in 繁體中文:

```
1. ✅ 正確 / ❌ 還有錯
   你的：<user rewrite>
   參考：<corrected_text>
   <one short line: for ❌ name the error type still present; for ✅ nothing>
2. ...

結果：<passed>/<count>
```

   - Nothing else. No preamble, no extra tips.

## Edge cases

- The user answers only some items: grade those and list the unanswered ones as 未作答.
- A tool returns an error: show the message as-is and tell the user the backend may not be running (`docker compose up -d`).
