# dotfiles

My Mac development environment, version-controlled for reproducibility.

## What's tracked

| Package | Config for |
|---------|-----------|
| `zsh/` | `.zshrc`, `.zprofile` — Oh My Zsh, plugins, aliases |
| `github/` | `.gitconfig` — git user config |
| `ghostty/` | Ghostty terminal — theme, keybindings, splits |
| `ohmyposh/` | Oh My Posh prompt — Catppuccin theme |
| `micro/` | Micro editor — 8 Catppuccin colorschemes |
| `nvim/` | Neovim — LazyVim setup |
| `yazi/` | Yazi file manager — config + theme |
| `zed/` | Zed editor — settings |
| `vscode/` | VS Code — settings, keybindings |
| `Brewfile` | All Homebrew formulae and casks |

## Setting up a new machine

See **[new-machine.md](./new-machine.md)** for the complete step-by-step guide (also usable as LLM instructions for one-shot setup).

## Applying on an existing machine

Requires [GNU stow](https://www.gnu.org/software/stow/):

```bash
cd ~/dotfiles
stow --target="$HOME" zsh github ghostty ohmyposh micro nvim yazi zed vscode
```
