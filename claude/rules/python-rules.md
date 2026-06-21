---
paths:
  - "**/*.py"
  - "**/*.pyi"
---

<!--
Maintainer note (stripped from Claude's context):
Path-scoped Claude Code rule. The `paths:` frontmatter means this loads only when
Claude touches Python files, so it doesn't cost context on non-Python work.

Captures the non-obvious decisions beyond what Ruff (lint + format) enforces.

To reuse across repos, symlink it into a rules dir:
  ln -s "$PWD/claude/rules/python-rules.md" ~/.claude/rules/python-rules.md      # every repo on this machine
  ln -s /path/to/laptop/claude/rules/python-rules.md <repo>/.claude/rules/        # one repo

Reconcile with ~/.claude/CLAUDE.md: that file says format with Ruff, whose default
line length is 88, while these rules say 80. Pick one and set `line-length` in your
Ruff config so the formatter and this rule agree — the body states 80, so edit it if
you standardize on 88.
-->

# Python style

Idiomatic Python conventions beyond what `ruff check` and `ruff format` enforce.

## Guiding principle

Optimize for the reader, not the writer. **Be consistent** — match the surrounding file. Local
style fills only gaps the guide is silent on, and never justifies expanding an existing deviation.

## Formatting

Ruff handles most of this; the non-default choices:

- **80-character lines.** Never use backslash continuation — rely on implicit joining inside `()`, `[]`, `{}`. Allowed to overrun: long imports, URLs/paths in comments, long string constants, `# noqa`/`# pylint:` directives.
- Add a **trailing comma** after the last element only when the closing `)`/`]`/`}` is on its own line (and for one-element tuples) — it's the signal that makes the formatter break one-item-per-line.
- No spaces around `=` for keyword args and defaults — **unless** the parameter has a type annotation, then add them: `def f(a: int = 0)`, but `f(a=0)`.
- Parenthesize one-element tuples for clarity (`(x,)`); otherwise use parentheses sparingly.

## Imports

- Import **modules, not names**: `from pkg import module`, then call `module.func()`. Don't import individual functions/classes. Exception: symbols from `typing` and `collections.abc`.
- Use **full package paths**; never relative imports.
- `import x as z` only for a standard abbreviation (`import numpy as np`). `from x import y as z` to resolve a name clash, a too-generic name, or an unwieldy one.
- One import per line, at file top after the module docstring. Separate groups with a blank line, each sorted case-insensitively by full path: (1) `from __future__`, (2) stdlib, (3) third-party, (4) local/sub-package.

## Naming

- `module_name`, `package_name`, `ClassName`, `ExceptionName`, `method_name`, `function_name`, `GLOBAL_CONSTANT_NAME`, `global_var_name`, `instance_var_name`, `function_parameter_name`, `local_var_name`. Modules are `lower_with_under.py` (never dashes); classes and exceptions are `CapWords`; exception names end in `Error`.
- `_single_leading_underscore` marks protected/module-internal. Avoid `__double_leading` name-mangling — it hurts readability and testability. Never define your own `__dunder__` names.
- Avoid single-char names except counters (`i`/`j`), `e` in `except`, `f` for a file handle, and math notation that cites its source. Don't encode type in the name (`names`, not `name_list` or `id_to_name_dict`).
- Descriptive over abbreviated; keep related classes and functions together in one module.

## Comments & docstrings

- `"""Triple double quotes."""` The summary is one line ≤80 chars ending in punctuation; for multi-line, leave a blank line after the summary.
- A docstring is **mandatory** for any function that is public API, nontrivial, or non-obvious — enough to call it without reading the body. Sections use a 2–4 space hanging indent: **Args:**, **Returns:** (or **Yields:** for generators; omit if it only returns `None`), **Raises:**.
- Classes get a docstring; document public attributes in an **Attributes:** section.
- Pick imperative ("Fetch…") or descriptive ("Fetches…") and stay consistent within a file. An `@override` method needs a docstring only if it refines the base behavior.
- Comment the *why* and the tricky, never restate the code. Inline comments start ≥2 spaces from the code: `# ` then a capitalized phrase.

## Exceptions

- Raise built-ins where they fit (`ValueError`, `TypeError`); define custom exceptions subclassing `Exception` (or a closer built-in), name ending in `Error`.
- Never bare `except:` or catch broad `Exception` except to log-and-reraise or at a top-level boundary. Keep the `try` body minimal; put cleanup in `finally` or a `with`.
- Don't use `assert` for control flow or validating public input — it's for internal invariants only (stripped under `-O`).

## Functions & arguments

- **Never use a mutable default** (`def f(x=[])`). Default to `None` and build the real value inside.
- Keep functions small and single-purpose; reconsider any over ~40 lines.
- Lambdas only for one-liners (<60–80 chars); otherwise a named nested function. Prefer the `operator` module to lambdas for common ops.
- Conditional expressions (`a if cond else b`) only when each part is simple and fits one line.
- Properties only for cheap, unsurprising computed attribute access; otherwise use explicit methods. Add getters/setters only when get/set carries real logic or cost — otherwise expose a plain public attribute.

## Control flow & truthiness

- Use implicit truthiness (`if seq:`, `if not seq:`); never compare to `True`/`False`/`[]`/`0` with `==`. But use `if x is None:` for None checks, and explicit `== 0` for numbers where 0 is meaningful.
- Use default iterators and operators: `for k in d`, `if k in d` — not `d.keys()` / `d.has_key()`.
- Comprehensions only for simple cases — no multiple `for` clauses and no multiple filters; fall back to a loop.

## Modules, globals & main

- **Avoid mutable global state.** Module level should hold constants (`CAPS_WITH_UNDER`). If a mutable global is truly needed, prefix `_` and document why.
- Nested/inner functions and classes are fine for closing over locals, not merely for hiding — prefix `_` at module level instead, which keeps them testable.
- Put real work in a `main()` and guard with `if __name__ == '__main__':`; no top-level code with side effects (it runs on import and breaks `pydoc`/tooling).
- Manage resources with `with` (or `contextlib.closing`); explicitly close files and sockets. Don't rely on `__del__`.

## Strings

- Format with f-strings, `%`, or `str.format` — choose per case; use `+` only for simple concatenation, never for formatting. Pick `'` or `"` per file and switch only to avoid escaping. Multi-line uses `"""` with `textwrap.dedent`.
- **Logging:** pass a literal format string plus args (`logger.info("got %s", x)`), not a pre-built f-string — this lets handlers defer and aggregate formatting.
- **Error messages:** make them match the condition and keep interpolated values greppable — `{x=}` in f-strings, or `%r`.

## Type annotations

- Encouraged; required at minimum for public APIs and for error-prone or complex code. Check with a type checker (mypy/pytype).
- Use built-in generics and abstract types: `list[int]`, `dict[str, int]`, `collections.abc.Sequence` over `typing.List` / concrete types. Import typing symbols directly (`from collections.abc import Sequence`).
- **Explicit `X | None`** — write `a: str | None = None`, never `a: str = None`.
- Don't annotate `self`/`cls`, or `__init__`'s `None` return. Annotate hard-to-infer assignments (`a: Foo = make()`); don't use old `# type:` comments. `tuple[int, ...]` for homogeneous, `tuple[int, str]` for fixed.
- Forward references via `from __future__ import annotations` (or string quotes). Type-only imports go under `if TYPE_CHECKING:`. `TypeVar`/`ParamSpec` for generics; `AnyStr` when a function's string types must match.

## Misc

- **Avoid "power features"** — metaclasses, bytecode access, dynamic `__class__`/inheritance, custom `__del__`, import-time reflection tricks. They make code harder to read and maintain.
- **Threading:** don't rely on the atomicity of built-in types. Use `queue.Queue` for inter-thread hand-off; otherwise use `threading` locks, preferring condition variables.
- **TODOs:** `# TODO: <link-or-bug> - explanation` — reference an issue, not a person.
