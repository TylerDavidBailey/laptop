---
paths:
  - "**/*.go"
---

<!--
Maintainer note (stripped from Claude's context):
Path-scoped Claude Code rule. The `paths:` frontmatter means this loads only when
Claude touches Go files, so it doesn't cost context on non-Go work.

Captures the non-obvious decisions beyond what gofmt and the compiler enforce.

To reuse across repos, symlink it into a rules dir (you said you'd wire this up):
  ln -s "$PWD/claude/rules/go-rules.md" ~/.claude/rules/go-rules.md      # every repo on this machine
  ln -s /path/to/laptop/claude/rules/go-rules.md <repo>/.claude/rules/   # one repo

Reconcile with ~/.claude/CLAUDE.md when you clean it up: these rules keep test
validation inline / use cmp.Diff and avoid assertion libraries, but CLAUDE.md
prefers testify. Pick a winner so the two files don't contradict each other.
-->

# Go style

Idiomatic Go conventions beyond what `gofmt` and the compiler enforce.

## Guiding principles

Apply in priority order — an earlier principle wins when they conflict:

1. **Clarity** — write for the reader, not the author. Clarity over cleverness. Comments explain *why*, not *what*.
2. **Simplicity** — the simplest thing that works; reach for standard constructs before inventing abstraction.
3. **Concision** — high signal-to-noise; cut repetition and opaque names.
4. **Maintainability** — easy for the next person to change correctly; don't hide critical details.
5. **Consistency** — match the surrounding package. Local style fills only gaps the guide is silent on, and never justifies expanding an existing deviation.

## Formatting

- All code must be `gofmt`-clean; use `goimports` for import grouping. Never hand-format.
- No line-length limit — refactor rather than wrap. Don't split a line just to fit a string or before an indentation change.
- Keep function signatures on one line; if args overflow, use an option struct or factor out locals.

## Naming

- `MixedCaps` / `mixedCaps`, never `snake_case` or `SCREAMING_SNAKE_CASE` — including constants (`MaxPacketSize`, not `MAX_PACKET_SIZE`). Name constants by role, not value.
- Initialisms keep a single case: `URL`, `userID`, `HTTPServer` — never `Url`, `userId`, `HttpServer`.
- **Packages:** short, lowercase, single word, no underscores, no plurals (`tabwriter`, not `tab_writer`). Never `util`, `common`, `helper`, `base`, `model` — name what the package provides. Group related code by domain across files; don't enforce one-type-per-file.
- **Don't stutter:** `widget.New`, not `widget.NewWidget`; don't repeat the package or a known type in a symbol name.
- **Receivers:** 1–2 letters abbreviating the type (`func (t *Tray)`), consistent across every method on the type; omit if unused.
- **Variables:** length proportional to scope, inverse to frequency of use. Single letters only for indices/coords/receivers. Drop type qualifiers (`users`, not `userSlice`; `userCount`, not `numUsers`). Don't shadow stdlib package names.
- **Getters:** no `Get` prefix — `Counts()`, not `GetCounts()`. Use `Fetch`/`Compute` when the call is expensive or can block. Noun-like names for value-returning funcs, verb-like for action funcs.

## Comments & docs

- Every exported symbol gets a doc comment: a full, capitalized, punctuated sentence starting with the name — `// Encode writes ...`.
- The package comment sits directly above `package` (no blank line), one per package. For `main`: `// The foo command ...`.
- Document the non-obvious: error-prone params, returned sentinel errors/types, cleanup requirements (`Call Stop() to release ...`), and non-standard context/concurrency behavior. Skip the obvious.

## Declarations & initialization

- `:=` for non-zero initialization inside functions; `var` for the zero value or a "ready to use" empty (`var buf bytes.Buffer`, `var s []string`).
- Prefer a `nil` slice over `[]T{}`; don't distinguish nil from empty in APIs — test with `len(s) == 0`.
- Preallocate with `make(..., 0, n)` only when the size is known; `new(pb.Bar)` for a pointer to a zero value (common with protos).
- Omit zero-value fields from struct literals; use field names for structs from other packages.

## Control flow

- Handle errors immediately and return early; keep the happy path left-aligned.
- No `else` after a `return`. Don't split `if`/`switch` conditions across lines — extract a boolean local when one gets complex.

## Errors

- Error strings are lowercase with no trailing punctuation (unless they start with a proper noun or exported name).
- `error` is always the last return; return the `error` interface, not a concrete type; `nil` means success.
- Never signal errors in-band (`-1`, `""`, `nil`) — return an extra value: `Lookup(k) (string, bool)`.
- Add context by wrapping: `fmt.Errorf("loading config: %w", err)`. Put `%w` at the end so the chain reads outer→inner. Add meaning, don't restate the wrapped error.
- Use `%w` when callers may inspect the cause; use `%v` to deliberately sever the chain at a trust/system boundary (RPC, storage) so internals don't leak.
- Inspect with `errors.Is` (sentinel) or `errors.As` (type) — never string-match. Define sentinels (`var ErrFoo = errors.New(...)`) or custom types for programmatic handling.
- Don't log-and-return — log once at the boundary where the error stops propagating and let the caller decide. Reserve the error log level for actionable problems; gate expensive log payloads behind a level check.
- Don't `panic` or `log.Fatal` for normal failures (and no `log.Fatal` in libraries). Don't discard errors with `_` unless documented safe.

## Functions

- **Option struct** when many callers set options, options are shared across functions, or per-field docs help.
- **Variadic functional options** when most callers need none, there are many, or each takes multiple params. Options should take arguments (`FailFast(true)`, not `EnableFailFast()`) and may return `error` to validate before applying.
- Accept interfaces, return concrete types.
- Name result params only when the caller must act on them or they're set in a `defer`.

## Context

- `context.Context` is the first parameter, named `ctx`. Never store it in a struct; pass it explicitly through the call chain.
- `context.Background()` only in `main`, `init`, or test setup — otherwise receive it from the caller.
- Never define custom context types.

## Concurrency

- Don't start a goroutine without a clear stop condition — drive exit with `context`, `sync.WaitGroup`, or a signal channel. Don't leak goroutines.
- Prefer synchronous APIs and let the caller add concurrency.
- Constrain channel direction in signatures (`<-chan T`, `chan<- T`) to clarify ownership.

## Interfaces & generics

- Define an interface where it's consumed, not where the concrete type lives. Keep interfaces small.
- Add an interface only for a real need (multiple implementations or a test seam) — start concrete, generalize later. Same for generics: reach for them only when concrete code genuinely repeats across types.

## Panics, Must, globals

- Reserve `panic` for impossible states or API misuse caught in review; never let it cross a package boundary — recover at the public edge and return an `error`. Don't recover to hide bugs.
- `MustXyz` helpers panic on failure — only for package init or tests, never on user input.
- Avoid mutable globals; inject dependencies. Keep `init()` for package setup with no side effects in libraries. Declare flag/`var` groups right after imports using `var`, not `:=`.

## Receivers, copying, misc

- Pointer receiver if the method mutates, the struct is large, or it holds a non-copyable field (`sync.Mutex`); value receiver for small, built-in-like types. Keep one receiver style across the type.
- Don't copy a struct containing a `sync.Mutex` or other non-copyable field.
- `crypto/rand` for anything security-sensitive — never `math/rand`.
- `%q` for quoted strings, `%v` for general values, `%+v` for structs in test output. Prefer `any` over `interface{}`.

## Testing

- Table-driven: a slice of case structs (`name`, inputs, `want`) run via `t.Run(tc.name, ...)` subtests.
- Failure messages give got-before-want with the inputs: `YourFunc(%v) = %v, want %v`.
- Compare complex values with `cmp.Diff` / `cmp.Equal`; keep validation logic inline rather than hiding it in assertion libraries.
- `t.Helper()` in helpers; `t.Fatal` when continuing is meaningless, `t.Error` to keep checking. Never call `t.Fatal` from a spawned goroutine — use `t.Error` then `return`.
- `t.Cleanup(...)` for teardown, over `defer`.
