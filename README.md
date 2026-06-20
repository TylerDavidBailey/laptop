# laptop

My macOS setup — shell config, Homebrew packages, and tooling.

## Contents

- `zsh/zshrc` — Zsh configuration
- `brew/Brewfile` — Homebrew packages and casks
- `claude/GLOBAL_CLAUDE.md` — Global Claude Code instructions (link to `~/.claude/CLAUDE.md`)

## Usage

```sh
# Symlink the shell config
ln -s "$PWD/zsh/zshrc" ~/.zshrc

# Install everything from the Brewfile
brew bundle --file=brew/Brewfile
```
