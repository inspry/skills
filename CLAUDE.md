# CLAUDE.md

Repo = Claude Code plugin marketplace of WordPress PHP skills for Inspry.

## Structure

- `.claude-plugin/marketplace.json` — marketplace; lists plugins.
- `plugins/inspry-php/.claude-plugin/plugin.json` — plugin manifest.
- `plugins/inspry-php/skills/<name>/SKILL.md` — one skill each; YAML frontmatter
  (`name`, `description`) + body. Templates/scripts in a subdir (e.g. `reference/`).

## Conventions

- All skills WordPress-focused (Inspry = WordPress agency).
- Assume **hollow** repos: no `wp-cli`, no DB, no live WP. Static analysis only.
- `description` field drives skill activation — make it specific, list triggers.
- Bump version in both `plugin.json` and `marketplace.json` on release.
- Test a change locally: `/plugin marketplace add ./` (path to this repo), install,
  verify skill activates.
