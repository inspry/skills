# Inspry Skills

Claude Code skills for [Inspry](https://github.com/inspry)'s WordPress PHP
workflows. Distributed as a Claude Code plugin marketplace so the whole team gets
the same skills and updates with one command.

## Install

In Claude Code, add the marketplace, then install the plugin:

```
/plugin marketplace add inspry/inspry-skills
/plugin install inspry-php@inspry-skills
```

That's it. Restart Claude Code if prompted. To update later:

```
/plugin marketplace update inspry-skills
```

## What's included

Everything ships in one plugin, **`inspry-php`**.

| Skill        | Status  | What it does                                                    |
| ------------ | ------- | -------------------------------------------------------------- |
| `php-lint`   | ✅ Ready | Check WordPress PHP for syntax/parse errors with `php -l`.      |
| `php-review` | 🔜 Soon | WordPress-focused PHP code review.                              |
| `php-security` | 🔜 Soon | Security audit for WordPress PHP (escaping, nonces, sanitize). |
| `php-best-practices` | 🔜 Soon | WordPress PHP best-practices guidance.               |

Skills activate automatically when your request matches. You don't invoke them
manually.

## Repo layout

```
.
├── .claude-plugin/
│   └── marketplace.json        # marketplace definition
└── plugins/
    └── inspry-php/
        ├── .claude-plugin/
        │   └── plugin.json      # plugin manifest
        └── skills/
            └── php-lint/
                └── SKILL.md     # skill instructions
```

## Contributing a skill

1. Add a dir under `plugins/inspry-php/skills/<skill-name>/`.
2. Write `SKILL.md` with YAML frontmatter (`name`, `description`). The
   `description` is what Claude uses to decide when to activate the skill, so make
   it specific and list trigger scenarios.
3. Put any templates/scripts the skill references in a subdir (e.g. `reference/`).
4. Bump the version in `plugin.json` and `marketplace.json`.
5. Open a PR.

## Notes

- Inspry WordPress repos are usually **hollow** (no live instance, no `wp-cli`, no
  DB). Skills assume static analysis and don't try to boot WordPress.
- Deploys are driven by GitHub Actions: `staging` branch → staging, `main` → live.
