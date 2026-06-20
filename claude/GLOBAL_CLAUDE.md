# Global Claude Instructions

Personal preferences that apply to all projects. Keep this file short — long instruction files get ignored.

## Git

- **YOU MUST** write commit messages using Conventional Commits: `<type>[optional scope]: <description>`.
  Use types like `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`. Mark breaking changes with `!` (e.g. `feat!:`) or a `BREAKING CHANGE:` footer.
- **YOU MUST** name branches using Conventional Branch: `<type>/<description>`.
  Use types like `feature`, `bugfix`, `hotfix`, `release`, `chore`. Use only lowercase letters, numbers, and hyphens to separate words (no spaces, underscores, or uppercase); no leading/trailing or consecutive hyphens. Dots are allowed only in release versions (e.g. `release/v1.2.0`).
- **YOU MUST NOT** commit directly to the default branch (`main`/`master`). Create a `<type>/<description>` branch first, then open a PR.

## Go

Write idiomatic Go (per Effective Go).

- **Lint and format with `golangci-lint`** (`golangci-lint run`, `golangci-lint fmt`) — tabs, braces on the same line. Never hand-format. Fix reported issues before considering work done.
- **Naming:** `MixedCaps`/`mixedCaps`, never underscores. Short lowercase package names; don't repeat the package name in exported identifiers (`bufio.Reader`, not `bufio.BufReader`). Getters omit `Get` (`obj.Owner()`); single-method interfaces take the `-er` suffix (`Reader`, `Writer`).
- **Errors:** Return `error` as the last value and handle it explicitly — no silent discards. Use the comma-ok idiom for map lookups and type assertions. Wrap with `fmt.Errorf("...: %w", err)` to add context. Word messages as the action being attempted, lowercase, no trailing punctuation or "failed"/"could not" prefixes (`connecting to db`, not `failed to connect to DB`) so they read as a narrative when stacked.
- **Control flow:** Prefer `if err != nil { return err }` early returns; use the `if`/`switch` initializer form. Reach for `range` to iterate.
- **`defer`** cleanup (`defer f.Close()`) right after acquiring the resource.
- **Allocation:** `make` for slices/maps/channels, `new`/`&T{}` for everything else. Design types so the zero value is usable; avoid needless `Init()`.
- **Concurrency:** "Share memory by communicating." Prefer channels over shared state with locks; keep goroutine lifetimes and ownership clear.
- **Logging:** Use zerolog (`github.com/rs/zerolog`). Prefer structured, leveled logging over `fmt`/`log`. Dependency-inject the logger rather than using the global one where it makes sense. Log an error once at the boundary (handler/`main`) where it stops propagating — don't log and return the same error at every level.
- **Doc comments** precede the declaration with no blank line and start with the name being documented.
- **Tests:** Use testify (`github.com/stretchr/testify`) — `require` for assertions that should halt the test, `assert` for ones that can continue. Write table-driven tests with `t.Run` subtests. Run tests in parallel with `t.Parallel()` where it's safe.

## Python

- **Use `uv`** for everything: `uv run`, `uv add`/`uv remove` for dependencies, `uv sync` to install, `uv venv` for environments. Don't use `pip`, `pipenv`, `poetry`, or `python -m venv` directly.
