# english-tutor-plugin

Claude Code plugin for the [english-tutor](https://github.com/LeshLiao/english-tutor) project. Bundles the skills that check English sentences and record mistakes through the `english-practice` MCP server.

**This plugin is useless on its own.** It needs an MCP server named `english-practice` registered in Claude Code (stdio or HTTP). The server and its backend live in the english-tutor repo, not here, because the stdio command contains a machine-specific path.

## Install

```bash
claude plugin marketplace add LeshLiao/lesh-marketplace     # once, if not already added
claude plugin install english-tutor-plugin@lesh-marketplace
```

Then register the MCP server (stdio, local development; adjust the path):

```bash
claude mcp add english-practice -- /opt/homebrew/bin/uv --directory ~/sourceCode/english-tutor/mcp-server run server.py
```

Restart Claude Code. Skills load at session start.

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

MCP tools used: `list_error_types`, `save_sentence`, `get_error_stats`, `search_sentences`, `get_recent_mistakes` (english-tutor MCP server ≥ step 07). `dismiss_sentence` is called directly when the user says a record was a false positive; no skill is needed.

## Layout

```
english-tutor-plugin/
├── .claude-plugin/plugin.json
├── skills/
│   ├── english-check/SKILL.md
│   ├── english-stats/SKILL.md
│   └── english-review/SKILL.md
└── README.md
```

## Changelog

- **1.1.0** (2026-11-02): `english-check` now calls `get_recent_mistakes` and flags repeated error types; new skills `english-stats` and `english-review`. Requires the step-07 MCP tools.
- **1.0.0** (2026-11-02): initial release with `english-check` (moved out of the english-tutor repo's `.claude/skills/`).
