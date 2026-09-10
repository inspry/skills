---
name: php-lint
description: Check WordPress PHP for syntax (parse) errors using PHP's built-in linter (php -l). Use when linting PHP, checking for syntax/parse errors, validating PHP before deploy, or catching fatal errors in a WordPress theme, plugin, or mu-plugin.
---

# PHP Syntax Linting (php -l)

Catch PHP **syntax / parse errors** with PHP's built-in linter, `php -l`. This is
a fast static check that never executes the code, so it's safe on any file. It
does **not** check coding style (Inspry doesn't enforce WPCS) — it only tells you
whether the file will parse.

## Context: Inspry repos are usually hollow

Most Inspry WordPress repos are **not** a live install: no `wp-cli`, no database,
no WordPress core. `php -l` needs none of that — it lints a single file in
isolation. Do not try to boot WordPress or run `wp`.

## Workflow

### 1. Confirm PHP is available

```bash
php -v 2>/dev/null || echo "php CLI missing"
```

If missing, tell the user; they need the `php` CLI installed to lint. Match the
major version to the client's target where it matters (parse rules differ across
PHP versions — e.g. `enum`, named args, `match`).

### 2. Pick the target — default is modified files

**Default: lint only the PHP files the user changed**, not the whole tree. Their
edits are what matters; a whole-tree scan is slow and surfaces pre-existing parse
errors in code they never touched (vendored plugins, legacy themes), which is
noisy and not actionable. Only scan the whole tree when the user explicitly asks
("lint everything") or when the repo isn't a git repo.

In a git repo, collect changed `.php` files — both uncommitted work and commits on
the current branch not yet in the live base — de-duplicated, existing files only.
The base is `main` (live) when present, else `staging`; comparing to it catches
work that isn't live yet, which is the point of a pre-deploy lint:

```bash
BASE=main; git rev-parse --verify -q main >/dev/null 2>&1 || BASE=staging
{ git diff --name-only --diff-filter=d HEAD;          # unstaged + staged vs HEAD
  git diff --name-only --diff-filter=d "$BASE"...HEAD; # commits not yet in base
} 2>/dev/null | sort -u | grep '\.php$'
```

- No output → no changed PHP files; tell the user, nothing to lint.
- Not a git repo, or user asked for a full scan → use the whole-tree command below.

Inspry repos deploy `staging`→staging and `main`→live, so `main`/`staging` cover
almost every repo. Adjust `BASE` if a client uses something else (e.g. `master`).

**Announce the target before linting.** State plainly which files you're about to
lint and why, so the user can catch a wrong scope. One line, e.g.:

> Linting 3 changed PHP files (uncommitted + commits vs `main`): `foo.php`,
> `bar.php`, `baz.php`. Not scanning the whole tree — say "lint everything" for that.

If you fall back to a whole-tree scan (not a git repo, or the user asked), say so
and why. If the selection is empty, say there are no changed PHP files to lint.

### 3. Lint the target files

Feed the selected files to `php -l`, print only failures:

```bash
BASE=main; git rev-parse --verify -q main >/dev/null 2>&1 || BASE=staging
{ git diff --name-only --diff-filter=d HEAD;
  git diff --name-only --diff-filter=d "$BASE"...HEAD;
} 2>/dev/null | sort -u | grep '\.php$' \
| xargs -r -n1 -P8 sh -c 'php -l "$1" 2>&1 || true' _ \
| grep -v '^No syntax errors detected'
```

The `sh -c '... || true'` wrapper matters: `php -l` exits **255** on a parse
error, and `xargs` treats status 255 as fatal and aborts the whole run, so a
broken file would stop the remaining files from being linted. Swallowing the exit
code keeps every file checked; failures still show via their printed output.

Or lint one file directly:

```bash
php -l path/to/file.php
```

- Exit code `0` → `No syntax errors detected`.
- Non-zero → prints `PHP Parse error: ... in <file> on line N`.

Empty output from the batch = everything parses. Any lines shown are the failing
files. `-P8` runs 8 in parallel; drop it if it muddles output ordering.

### 3b. Whole-tree scan (opt-in)

Only when explicitly requested or outside a git repo. Lint every `.php` file, skip
vendor/build:

```bash
find . -type f -name '*.php' \
  -not -path '*/vendor/*' \
  -not -path '*/node_modules/*' \
  -not -path '*/build/*' \
  -print0 \
| xargs -0 -r -n1 -P8 sh -c 'php -l "$1" 2>&1 || true' _ \
| grep -v '^No syntax errors detected'
```

### 4. Fix errors

`php -l` reports the file and line of the first parse error per file. Open it, fix
the syntax (unclosed brace/bracket/paren, missing `;`, stray token, bad
heredoc/nowdoc, PHP-version-incompatible syntax), then re-lint that file to
confirm. Parse errors are not auto-fixable — fix by hand.

## Reporting back

After a run, state plainly: **what target was linted and why** (changed files vs
whole tree, and the base branch used for the diff), how many files checked, how
many passed, and list each failing file with its `file:line` and the parse-error
message. If everything parses, say so.
