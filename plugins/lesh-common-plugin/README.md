# lesh-common-plugin

Personal Claude Code plugin by Lesh Liao. Bundles everyday skills, one agent, and three MCP servers (Heptabase, Context7, GitHub).

## Install from Github

```bash
claude plugin marketplace add LeshLiao/lesh-marketplace
claude plugin install lesh-common-plugin@lesh-marketplace
```

# After install:
   /Users/yourname/.claude/plugins/marketplaces/lesh-marketplace

Restart Claude Code afterwards. Plugins and skills load at session start.

## Contents

### Skills

| Command | What it does |
|---|---|
| `/lesh-common-plugin:create-github-issue <problem>` | Investigates the problem in the codebase first (root cause + solution), then creates a concise GitHub issue. Asks which repo via popup menu if not specified. |
| `/lesh-common-plugin:git-commit-one-line` | Reads the git diff and commits with a concise one-line English message. |
| `/lesh-common-plugin:git-commit-three-bullet-point` | Reads the git diff and commits with a three-bullet English message. |
| `/lesh-common-plugin:note-to-heptabase <memo>` | Writes the memo as a Heptabase card, places it on the "Note" whiteboard, and tags it only if an existing tag clearly fits. |
| `/lesh-common-plugin:taichung-weather-call-agent` | Manually runs the `taichung-weather` agent in a forked context. Slash-command only; the model never auto-invokes it. |

### Agent

- **taichung-weather**: reports today's weather for Taichung, Taiwan. Triggers on questions like "台中今天天氣" or "Taichung weather today". Uses web search only, runs on Haiku.

### MCP servers

- **context7**: `https://mcp.context7.com/mcp` (HTTP). Up-to-date library docs and code examples for any framework. No login needed; ask Claude to "use context7" when you want current API docs. Optional: an API key from context7.com raises rate limits, added as a `CONTEXT7_API_KEY` header in `.mcp.json`.
- **heptabase**: `https://api.heptabase.com/mcp` (HTTP). Needed by `note-to-heptabase`.
  On first use run `/mcp` and complete the OAuth login. Open the auth link in Edge:

  ```bash
  open -a "Microsoft Edge" "<auth url>"
  ```
- **github**: `https://api.githubcopilot.com/mcp/` (HTTP). Official GitHub MCP server — repos, issues, PRs, code search, etc. On first use run `/mcp` and complete the GitHub OAuth login.

## Layout

```
lesh-common-plugin/
├── .claude-plugin/plugin.json   # manifest (name, version, author)
├── .mcp.json                    # heptabase + context7 + github MCP servers
├── agents/taichung-weather.md
├── skills/
│   ├── create-github-issue/SKILL.md
│   ├── git-commit-one-line/SKILL.md
│   ├── git-commit-three-bullet-point/SKILL.md
│   ├── note-to-heptabase/SKILL.md
│   └── taichung-weather-call-agent/SKILL.md
└── README.md
```

## Adding a skill

1. Create `skills/<skill-name>/SKILL.md` with `name`, `description`, and optional `version` / `triggers` frontmatter.
2. Bump `version` in `.claude-plugin/plugin.json` and add a dated changelog entry: `- **x.y.z** (YYYY-MM-DD): what changed.`
3. Run `claude plugin validate .` from the marketplace root.
4. Reinstall or update the plugin, then restart Claude Code.

## Publish to GitHub

The git repository is the **marketplace root** (`~/sourceCode/marketplace/lesh-marketplace`), not this plugin folder. Claude Code registers a marketplace by repo, and it must find `.claude-plugin/marketplace.json` at the repo root.

### First-time push

1. Create an empty repo on GitHub named `lesh-marketplace` under the LeshLiao account. Do not add a README, .gitignore, or license there.

   ```bash
   open -a "Microsoft Edge" "https://github.com/new"
   ```

2. Initialise git at the marketplace root and make the first commit:

   ```bash
   cd ~/sourceCode/marketplace/lesh-marketplace
   git init -b main
   printf '.DS_Store\n' > .gitignore
   git add .
   git commit -m "Initial marketplace: lesh-common-plugin 1.2.0"
   ```

3. Connect the remote and push. This machine reaches the LeshLiao account through the SSH alias `github_Lesh`, so use that host instead of `github.com`:

   ```bash
   git remote add origin git@github_Lesh:LeshLiao/lesh-marketplace.git
   git push -u origin main
   ```

4. Register the GitHub marketplace in Claude Code and install the plugin:

   ```bash
   claude plugin marketplace add LeshLiao/lesh-marketplace
   claude plugin install lesh-common-plugin@lesh-marketplace
   ```

   If the repo is private, `claude plugin marketplace add` needs git access to it. Because the SSH alias is not `github.com`, use the explicit SSH URL instead:

   ```bash
   claude plugin marketplace add git@github_Lesh:LeshLiao/lesh-marketplace.git
   ```

5. Restart Claude Code. Check with `/plugin` that lesh-common-plugin is listed and enabled.

### Publishing an update

1. Edit the plugin, then bump `version` in `.claude-plugin/plugin.json` and add a dated changelog line below (`- **x.y.z** (YYYY-MM-DD): ...`).
2. Validate, commit, and push:

   ```bash
   cd ~/sourceCode/marketplace/lesh-marketplace
   claude plugin validate .
   git add .
   git commit -m "lesh-common-plugin 1.x.0: <what changed>"
   git push
   ```

3. Pull the new version into Claude Code and restart:

   ```bash
   claude plugin marketplace update lesh-marketplace
   claude plugin update lesh-common-plugin@lesh-marketplace
   ```

## Changelog

- **1.5.0** (2026-09-20): added `create-github-issue` skill (investigate first, popup repo menu, concise issue via `gh`).
- **1.4.0** (2026-09-19): added GitHub MCP server (official remote server, OAuth).
- **1.3.0** (2026-09-11): added `taichung-weather-call-agent` skill (slash command that runs the `taichung-weather` agent).
- **1.2.0** (2026-09-08): added Context7 MCP server.
- **1.1.0** (2026-09-08): added `note-to-heptabase` skill.
- **1.0.0** (2026-09-08): initial release with the two git-commit skills, `taichung-weather` agent, and Heptabase MCP config.
