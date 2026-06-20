# laptop

My macOS setup — shell config, Homebrew packages, and tooling.

## Contents

- `zshrc` — Zsh configuration
- `Brewfile` — Homebrew packages and casks

## Usage

```sh
# Symlink the shell config
ln -s "$PWD/zshrc" ~/.zshrc

# Install everything from the Brewfile
brew bundle
```
