---
paths:
  - "**/*.sh"
  - "**/*.bash"
---

<!--
Maintainer note (stripped from Claude's context):
Path-scoped Claude Code rule. The `paths:` frontmatter means this loads only when
Claude touches shell files, so it doesn't cost context on non-shell work.

To reuse across repos, symlink it into a rules dir:
  ln -s "$PWD/claude/rules/shell-rules.md" ~/.claude/rules/shell-rules.md     # every repo on this machine
  ln -s /path/to/laptop/claude/rules/shell-rules.md <repo>/.claude/rules/      # one repo

Scope caveats:
- These rules are Bash-only. This rule is scoped to *.sh / *.bash and deliberately
  does NOT cover this repo's zsh/zshrc — several rules ([[ ]], arrays, local, "$@")
  behave differently or don't apply in zsh.
- Path rules match by filename, so an extensionless executable (shebang #!/bin/bash,
  no .sh) won't auto-load this rule. Give such scripts a .sh source name, or paste
  the rule in when working on them.
-->

# Shell style

**Bash only** — use `#!/bin/bash`. Format with `shfmt` and lint with `shellcheck`.

## When to use shell at all

- Shell is for **small utilities and simple wrappers** only. If a script grows past **~100 lines**, gains non-trivial control flow, or needs performance, rewrite it in Python or Go.
- Optimize for the next maintainer, not the author. **Be consistent** — match the surrounding script first; the rules below fill the gaps it leaves.

## Files & invocation

- Shebang `#!/bin/bash` with minimal flags. **Libraries** must end in `.sh` and must not be executable; **executables** take `.sh` or no extension (prefer none on `PATH`).
- Filenames are `lower_case` with underscores, never dashes.
- **Never** set SUID/SGID on a script — use `sudo` for elevated access.

## Comments

- A **file header** describing the script's contents is required at the top.
- A **function header** is required for every library function and any non-obvious one — Description, `Globals:` (read/modified), `Arguments:`, `Outputs:`, `Returns:`.
- Comment only tricky or non-obvious code; don't narrate the obvious. TODOs use `# TODO(name): explanation`.

## Formatting

- **2-space indent, never tabs** (sole exception: the tab indentation required by `<<-` here-docs). **80-column** lines.
- `; then` and `; do` go on the **same line** as `if`/`for`/`while`/`until`; `else`, `fi`, `done` get their own line aligned with the opener. Write `for x in "$@"` explicitly.
- A pipeline that fits stays on one line; otherwise **one segment per line**, the `|` leading each continuation, indented 2 spaces, joined with `\`.
- `case`: indent alternatives 2 spaces, patterns unquoted; one-liners get a space after `)` and before `;;`, otherwise action and `;;` go on their own lines. Don't use `;&` / `;;&`.

## Quoting & expansion

- **Quote** anything containing a variable, command substitution, space, or shell metacharacter. Prefer `"${var}"`; brace all but single-char positionals (`"$1"` is fine).
- Pass args as `"$@"`, never `$*` (use `$*` only when deliberately joining into one string). Expand arrays quoted: `"${array[@]}"`.
- Integers and shell specials (`$?`, `$#`, `$$`, `$!`) needn't be quoted.

## Tests & arithmetic

- Use `[[ … ]]`, never `[ … ]`, `test`, or `/usr/bin/[` — it avoids word-splitting and pathname expansion. Use `==` (not `=`) for string equality.
- Test strings with quoted `-z` / `-n`: `[[ -z "${var}" ]]`, not filler comparisons.
- Numeric comparison and arithmetic use `(( … ))` / `$(( … ))` — never `let`, `expr`, `$[ … ]`, or `<` / `>` inside `[[ ]]` (those compare lexically). Drop the `${}` inside `$(( ))`. Beware a bare `(( i++ ))` under `set -e` — it returns non-zero when the result is 0.

## Variables & functions

- Declare function-local vars with `local`. **Separate declaration from a command-substitution assignment** — `local x; x="$(cmd)"` — because `local`'s own exit code would otherwise mask the command's.
- `lower_case` for locals and function names; `UPPER_CASE` for constants, globals, and exported env vars. Make constants `readonly` (or `declare -r`) at the top.
- Define functions as `name() {` (no `function` keyword unless used consistently project-wide), brace on the same line. Group all functions below the constants and above the first call.
- A script containing **any** function needs a `main()`, invoked as the last line: `main "$@"`.

## Commands & gotchas

- Command substitution is `$(…)`, never backticks (it nests cleanly: `$(cmd "$(inner)")`).
- **Check every return value** — `if ! cmd; then …`, or inspect `$?`. For pipelines use `PIPESTATUS`, copied immediately (`rc=( "${PIPESTATUS[@]}" )`) since the next command overwrites it.
- Prefer Bash builtins / parameter expansion over spawning tools — `"${str//foo/bar}"` over `sed`, `[[ $s =~ re ]]` over `grep`, `(( … ))` over `expr`.
- Use **arrays** for lists, not space-separated strings. Expand filenames as `./*`, not `*`, so names starting with `-` aren't read as flags.
- Don't pipe into `while` (the loop runs in a subshell and its variable changes vanish) — use `while read …; do …; done < <(cmd)` or `readarray`.
- **Never** use `eval` or shell `alias`es in scripts; use functions instead.
