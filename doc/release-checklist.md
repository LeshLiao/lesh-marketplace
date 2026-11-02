# Release checklist (plugin version bump)

This marketplace hosts several plugins under `plugins/<plugin-name>/`. Each plugin is versioned independently: bump only the plugin(s) whose files actually changed.

Every time you bump a plugin version or push a new version (e.g. 1.5.0), ALL of these must be updated to the same version before pushing:

1. `plugins/<plugin-name>/.claude-plugin/plugin.json` — bump `version`.
2. `plugins/<plugin-name>/README.md` — add a dated changelog entry: `- **x.y.z** (YYYY-MM-DD): what changed.`
3. Root `README.md` — update that plugin's row in the `Plugins` version table.
4. Commit, then `git push` (origin is SSH).

When adding a new plugin, also register it in `.claude-plugin/marketplace.json` and add a row for it in the root `README.md` table, starting at `1.0.0`.
