# Release checklist (plugin version bump)

Every time you bump the plugin version or push a new version (e.g. 1.5.0), ALL of these must be updated to the same version before pushing:

1. `plugins/lesh-common-plugin/.claude-plugin/plugin.json` — bump `version`.
2. `plugins/lesh-common-plugin/README.md` — add a dated changelog entry: `- **x.y.z** (YYYY-MM-DD): what changed.`
3. Root `README.md` — update the `Current version` line to the new version.
4. Commit, then push via SSH: `git push git@github.com:LeshLiao/lesh-marketplace.git main` (HTTPS origin fails).
