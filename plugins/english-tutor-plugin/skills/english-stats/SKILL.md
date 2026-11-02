---
name: english-stats
version: 1.0.0
description: Summarize which English error types the user makes most often, as a table with a one-line focus suggestion, using the english-practice MCP server. Invoke with /english-tutor-plugin:english-stats [period], e.g. "this week", "last 30 days", or nothing for all time.
argument-hint: [period such as "this week" or "last 30 days"]
allowed-tools:
  - mcp__english-practice__get_error_stats
  - mcp__english-practice__search_sentences
---

# english-stats

Show the user a short statistics summary of their recorded English mistakes.

## Steps

1. **Work out the date range** from the argument. Today's date is in your context.
   - No argument or "all" → no `from_date` / `to_date`.
   - "today", "this week" (Monday to today), "this month", "last 7 days", "last 30 days", an explicit date or month → convert to ISO `from_date` (and `to_date` only if the range ends before today).

2. **Call `get_error_stats`** with that range.

3. **If the list is empty**, reply with one line in 繁體中文 saying there are no recorded mistakes in that period, and stop.

4. **For the top 3 error types**, call `search_sentences` with `error_type` and `limit: 1` (in parallel) to fetch one latest example each.

5. **Reply** in 繁體中文 in exactly this layout:

```
期間：<range in words, e.g. 全部、本週（2026-10-27 起）>
錯誤統計（共 <total> 次）：

| # | 錯誤類型 | 次數 | 最近例子 |
|---|----------|------|----------|
| 1 | <error_type> | <count> | <error_snippet> → <suggestion> |
| 2 | ... | ... | ... |

建議：<one sentence in 繁體中文 naming the top error type and the single rule to watch for>
```

   - Keep error type names in English exactly as returned.
   - List every error type returned, but fill 最近例子 only for the top 3; leave the cell empty for the rest.
   - Nothing else. No preamble, no extra tips.

## Edge cases

- The argument cannot be understood as a period: say so in one line and ask for a clearer one. Do not call any tool.
- A tool returns an error: show the message as-is and tell the user the backend may not be running (`docker compose up -d`).
