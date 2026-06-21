---
paths:
  - "**/*.md"
  - "**/*.markdown"
---

<!--
Maintainer note (stripped from Claude's context):
Path-scoped Claude Code rule. The `paths:` frontmatter means this loads only when
Claude touches Markdown files, so it doesn't cost context on non-Markdown work.

To reuse across repos, symlink it into a rules dir:
  ln -s "$PWD/claude/rules/markdown-rules.md" ~/.claude/rules/markdown-rules.md     # every repo on this machine
  ln -s /path/to/laptop/claude/rules/markdown-rules.md <repo>/.claude/rules/         # one repo

Two reconciliations to make:
- These rules wrap prose at 80 columns; this repo's own Markdown (CLAUDE.md, these
  rule files) uses long unwrapped lines. The body states 80 — edit it if you'd rather
  not wrap. Either way the "match the surrounding file" principle wins on edits.
- `[TOC]` is a non-standard directive, not part of Markdown; it does NOT render on
  GitHub. The body marks it tool-specific — drop that line if your docs live on GitHub.
-->

# Markdown style

Conventions for clear, consistent Markdown.

## Guiding principles

- **Minimum viable documentation.** A small set of fresh, accurate docs beats a sprawling, stale one. Delete cruft often, in small batches.
- **Better is better than best.** Ship a useful improvement now and refine in follow-ups, rather than blocking on perfection.
- **Be consistent** — match the surrounding file. When the guide and an existing file disagree, keep the file consistent.

## Document layout

- One **H1** title per document, matching or nearly matching the filename, followed by a 1–3 sentence intro.
- Use **H2** and deeper for every other section; give each heading a unique, fully descriptive name.
- Optional `[TOC]` directive right after the intro — note this is a non-standard extension that won't render on plain GitHub.
- Optional "See also" section of links at the end.

## Headings

- **ATX style** (`#`, `##`, …), never underline style (`===` / `---`).
- One space after the `#`, and a blank line before and after each heading.

## Lists

- **Lazy numbering** — use `1.` for every item in long or likely-to-change ordered lists; spell out `1.`, `2.`, `3.` only for short, stable ones.
- **4-space indent** for nested items and wrapped lines: 2 spaces after `N.` for numbered, 3 after the bullet for unordered — text lands at column 4.
- Single-spaced (no blank line between items) is fine only for short, single-line, non-nested lists; otherwise separate items with blank lines.

## Code

- **Inline backticks** for code, filenames and file types (`README.md`), field names, and to escape characters from Markdown processing.
- **Fenced blocks** (```` ``` ````), never indented blocks, and **declare the language** on the fence.
- Inside a list, indent a code block to align with the item's text so it doesn't break the list. Continue long shell commands with a trailing `\`.

## Links

- **Informative link text** — wrap a natural phrase; never "here", "this link", or a bare URL.
- Prefer **explicit repo paths** (`[page](/path/to/page.md)`) over fully-qualified URLs. Relative links are safe only within the same directory — avoid `../` hops.
- Use **reference links** for long URLs or ones repeated several times; put the definition just before the next heading (or at document end if it's shared across sections).

## Images & tables

- Use images **sparingly**, prefer simple screenshots, and always provide alt text.
- Use **tables** only for data that benefits from scanning. Don't use them where a list reads better, and avoid tables with lopsided dimensions or paragraphs in cells; reference links keep cell width down.

## Markdown vs. HTML

- **Strongly prefer Markdown to HTML.** Avoid HTML hacks — they hurt readability and portability.

## Prose

- Wrap prose at **80 columns**; let links, tables, headings, and code blocks overrun (and still wrap the text around a link).
- **No trailing whitespace.** Break a line with a trailing `\` (sparingly), not with two trailing spaces.
- Preserve the real capitalization of product, tool, and binary names.
