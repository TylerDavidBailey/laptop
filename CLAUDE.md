# laptop

Personal macOS setup repo: shell config, Homebrew packages, and tooling.

## Important

- **This is a public repo. Never commit secrets** (tokens, keys, passwords).
  Secrets are sourced from `~/.zsh_secrets`, which is gitignored.

## Structure

- `zsh/zshrc` — Zsh configuration, symlinked to `~/.zshrc`
- `brew/Brewfile` — Homebrew formulae and casks (`brew bundle --file=brew/Brewfile`)
- `claude/GLOBAL_CLAUDE.md` — Global Claude Code instructions, linked to `~/.claude/CLAUDE.md`
