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

### 2. Lint a single file

```bash
php -l path/to/file.php
```

- Exit code `0` → `No syntax errors detected`.
- Non-zero → prints `PHP Parse error: ... in <file> on line N`.

### 3. Lint a whole tree

Lint every `.php` file, skip vendor/build, print only the failures:

```bash
find . -type f -name '*.php' \
  -not -path '*/vendor/*' \
  -not -path '*/node_modules/*' \
  -not -path '*/build/*' \
  -print0 \
| xargs -0 -n1 -P8 php -l 2>&1 \
| grep -v '^No syntax errors detected'
```

Empty output = everything parses. Any lines shown are the files with errors.
`-P8` runs 8 in parallel; drop it if it muddles output ordering on a failure.

### 4. Fix errors

`php -l` reports the file and line of the first parse error per file. Open it, fix
the syntax (unclosed brace/bracket/paren, missing `;`, stray token, bad
heredoc/nowdoc, PHP-version-incompatible syntax), then re-lint that file to
confirm. Parse errors are not auto-fixable — fix by hand.

## Reporting back

After a run, state plainly: how many files checked, how many passed, and list each
failing file with its `file:line` and the parse-error message. If everything
parses, say so.
