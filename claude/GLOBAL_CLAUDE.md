# Global Claude Instructions

<!--
Maintainer note (stripped from Claude's context): personal preferences that apply
to all projects (~/.claude/CLAUDE.md). Keep under 200 lines — longer files consume
more context and reduce adherence. Language-specific rules can move to
~/.claude/rules/*.md with `paths:` frontmatter so they load only for matching files.
-->

## Git

- **YOU MUST** write commit messages using Conventional Commits: `<type>[optional scope]: <description>`.
  Use types like `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`. Mark breaking changes with `!` (e.g. `feat!:`) or a `BREAKING CHANGE:` footer.
- **YOU MUST** name branches using Conventional Branch: `<type>/<description>`.
  Use types like `feature`, `bugfix`, `hotfix`, `release`, `chore`. Use only lowercase letters, numbers, and hyphens to separate words (no spaces, underscores, or uppercase); no leading/trailing or consecutive hyphens. Dots are allowed only in release versions (e.g. `release/v1.2.0`).
- **YOU MUST NOT** commit directly to the default branch (`main`/`master`). Create a `<type>/<description>` branch first, then open a PR.
- **PR/MR descriptions:** Check for a repo template first (e.g. `.github/PULL_REQUEST_TEMPLATE.md`, `.gitlab/merge_request_templates/`) and use it if present. Otherwise use:

  ```markdown
  ## What

  ## Why

  ## Testing
  ```

  If no testing was done, put a `TODO` under `## Testing`.

## Go

Write idiomatic Go (per Effective Go) — assume standard conventions; the rules below are the non-default choices.

- **Lint and format with `golangci-lint`** (`golangci-lint run`, `golangci-lint fmt`). Never hand-format. Fix reported issues before considering work done.
- **Error messages:** word them as the action being attempted, lowercase, no trailing punctuation or "failed"/"could not" prefixes (`connecting to db`, not `failed to connect to DB`) so they read as a narrative when stacked. Wrap with `fmt.Errorf("...: %w", err)` to add context.
- **Logging:** Use zerolog (`github.com/rs/zerolog`) — structured and leveled, not `fmt`/`log`. Dependency-inject the logger rather than the global one. Log an error once at the boundary (handler/`main`) where it stops propagating, not at every level.
- **Tests:** Use testify (`github.com/stretchr/testify`) — `require` for assertions that should halt the test, `assert` for ones that can continue. Write table-driven tests with `t.Run` subtests. Run in parallel with `t.Parallel()` where safe.
- **Mocks:** Generate with uber-go/mock (`go.uber.org/mock/gomock`, `go.uber.org/mock/mockgen`), not hand-written. Add a `//go:generate mockgen -destination=... -package=... -typed . <Interfaces>` directive (package mode, `-typed` for type-safe expectations) and regenerate with `go generate ./...`. In tests, `ctrl := gomock.NewController(t)` then `NewMockX(ctrl)` and `.EXPECT()` — `NewController(t)` registers cleanup, so no `defer ctrl.Finish()`.

## Python

- **Use `uv`** for everything: `uv run`, `uv add`/`uv remove` for dependencies, `uv sync` to install, `uv venv` for environments. Don't use `pip`, `pipenv`, `poetry`, or `python -m venv` directly.
- **Lint and format with Ruff** (`uv run ruff check --fix`, `uv run ruff format`). Never hand-format. Fix reported issues before considering work done.
