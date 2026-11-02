# english-tutor-plugin

Claude Code plugin for the [english-tutor](https://github.com/LeshLiao/english-tutor) project. Bundles the skills that check English sentences and record mistakes through the `english-practice` MCP server.

Since 1.2.0 the plugin is self-contained: it ships the `english-practice` MCP connection (`.mcp.json`, Streamable HTTP to the Lightsail deployment) and the `UserPromptSubmit` hook, so every project on every machine gets the same behaviour. The server and backend code live in the english-tutor repo.

## Install

```bash
claude plugin marketplace add LeshLiao/lesh-marketplace     # once, if not already added
claude plugin install english-tutor-plugin@lesh-marketplace
```

Then give Claude Code the API key of the deployed MCP server. `.mcp.json` expands `${ENGLISH_PRACTICE_API_KEY}`, so put it in `~/.claude/settings.json` (not in git):

```json
{ "env": { "ENGLISH_PRACTICE_API_KEY": "<API_KEY from /srv/english-tutor/.env on the server>" } }
```

Restart Claude Code. `/mcp` should show `plugin:english-tutor-plugin:english-practice` connected. Tool names carry the plugin prefix: `mcp__plugin_english-tutor-plugin_english-practice__save_sentence` etc.

If you still have a project-level `english-practice` server registered (`claude mcp list`), remove it with `claude mcp remove english-practice`, otherwise the tools appear twice.

## Hook

`hooks/hooks.json` registers `hooks/check_english.py` on `UserPromptSubmit`. For every prompt that looks like an English sentence (4–300 words, no CJK, no code, no secrets, not a `/` or `!` command) it spawns a background `claude -p` that checks the sentence and saves it with `source = hook`, and prints a one-line `✨ English hook: analysing in background: …` notice. Logs go to `~/.claude/english-tutor-hook/logs/` (override with `ENGLISH_HOOK_LOG_DIR`). It never blocks the conversation.

## Update

```bash
claude plugin marketplace update lesh-marketplace
claude plugin update english-tutor-plugin@lesh-marketplace
```

## Skills

| Command | What it does |
|---|---|
| `/english-tutor-plugin:english-check <sentence>` | Calls `list_error_types` and `get_recent_mistakes`, finds and classifies every mistake (reusing existing category names), saves the sentence with `save_sentence` (`source: skill`; correct sentences are saved too, with no errors), replies in a fixed 原句 / 修正 / 說明 / 錯誤 format, and flags error types the user repeated recently（最近第 N 次）. |
| `/english-tutor-plugin:english-stats [period]` | Calls `get_error_stats` (optionally for "this week", "last 30 days", …) and shows a table of error types, counts and one latest example each, plus a one-line focus suggestion. |
| `/english-tutor-plugin:english-review [error_type] [count]` | Calls `search_sentences`, shows a few past wrong sentences for the user to rewrite, then grades the rewrites against the stored corrections. |

MCP tools used (all prefixed `mcp__plugin_english-tutor-plugin_english-practice__`): `list_error_types`, `save_sentence`, `get_error_stats`, `search_sentences`, `get_recent_mistakes`. `dismiss_sentence` is called directly when the user says a record was a false positive; no skill is needed.

## Layout

```
english-tutor-plugin/
├── .claude-plugin/plugin.json
├── .mcp.json                 # english-practice over Streamable HTTP, key from ${ENGLISH_PRACTICE_API_KEY}
├── hooks/
│   ├── hooks.json            # UserPromptSubmit → check_english.py
│   └── check_english.py
├── skills/
│   ├── english-check/SKILL.md
│   ├── english-stats/SKILL.md
│   └── english-review/SKILL.md
└── README.md
```

## Changelog

- **1.2.0** (2026-11-02): ship the `english-practice` MCP connection (`.mcp.json`, HTTP to Lightsail, key via `${ENGLISH_PRACTICE_API_KEY}`) and the `UserPromptSubmit` hook; skills switch to the plugin-prefixed tool names.
- **1.1.0** (2026-11-02): `english-check` now calls `get_recent_mistakes` and flags repeated error types; new skills `english-stats` and `english-review`. Requires the step-07 MCP tools.
- **1.0.0** (2026-11-02): initial release with `english-check` (moved out of the english-tutor repo's `.claude/skills/`).
