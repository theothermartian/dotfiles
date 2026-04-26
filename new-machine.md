# New Mac Setup Guide

> **For LLMs:** This file is a complete, ordered instruction set to reproduce this Mac setup on a fresh machine. Execute every step in sequence. Each section includes verification commands — run them before moving on. If a step fails, diagnose before continuing; do not skip.

---

## Prerequisites

- macOS 13+ (Ventura or later), Apple Silicon (M-series) preferred
- Internet connection
- Apple ID signed in (for App Store apps if needed)
- This guide assumes a fresh user account with no prior setup

---

## Step 1 — Xcode Command Line Tools

Required by Homebrew and git.

```bash
xcode-select --install
```

Wait for the GUI installer to complete, then verify:

```bash
xcode-select -p
# Expected: /Library/Developer/CommandLineTools
```

---

## Step 2 — Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After install, add Homebrew to PATH (Apple Silicon):

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Verify:

```bash
brew --version
# Expected: Homebrew 4.x.x
```

---

## Step 3 — Clone dotfiles repo

```bash
git clone https://github.com/theothermartian/dotfiles ~/dotfiles
cd ~/dotfiles
```

Verify:

```bash
ls ~/dotfiles
# Expected: Brewfile  ghostty  github  micro  nvim  ohmyposh  README.md  vscode  yazi  zed  zsh  new-machine.md
```

---

## Step 4 — Install all packages from Brewfile

```bash
brew bundle --file=~/dotfiles/Brewfile
```

This installs: neovim, fzf, fd, ripgrep, bat, eza, yazi, micro, oh-my-posh, zoxide, fastfetch, gh, stow, tmux, btop, tldr, node, uv, JetBrains Mono font, Ghostty terminal, and more.

Verify a few key tools:

```bash
nvim --version | head -1
fzf --version
eza --version
ghostty --version
```

---

## Step 5 — Install Oh My Zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
```

This installs to `~/.oh-my-zsh`. Say yes if it asks to change your shell to zsh.

Verify:

```bash
ls ~/.oh-my-zsh
# Expected: oh-my-zsh.sh, plugins/, themes/, etc.
```

---

## Step 6 — Install zsh plugins

The `.zshrc` uses three plugins that are not bundled with Oh My Zsh:

```bash
ZSH_CUSTOM="${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}"

git clone https://github.com/zsh-users/zsh-syntax-highlighting \
  "$ZSH_CUSTOM/plugins/zsh-syntax-highlighting"

git clone https://github.com/zsh-users/zsh-autosuggestions \
  "$ZSH_CUSTOM/plugins/zsh-autosuggestions"

git clone https://github.com/marlonrichert/zsh-autocomplete \
  "$ZSH_CUSTOM/plugins/zsh-autocomplete"
```

Verify:

```bash
ls "$ZSH_CUSTOM/plugins/"
# Expected: zsh-syntax-highlighting  zsh-autosuggestions  zsh-autocomplete
```

---

## Step 7 — Apply dotfiles with GNU stow

Stow creates symlinks from the dotfiles repo into the home directory. Run it for each package:

```bash
cd ~/dotfiles

# Shell config (.zshrc, .zprofile → ~/)
stow --target="$HOME" zsh

# Git config (.gitconfig → ~/)
stow --target="$HOME" github

# Terminal, prompt, editor configs (→ ~/.config/)
stow --target="$HOME" ghostty
stow --target="$HOME" ohmyposh
stow --target="$HOME" micro
stow --target="$HOME" nvim
stow --target="$HOME" yazi
stow --target="$HOME" zed

# VS Code settings (→ ~/Library/Application Support/Code/User/)
stow --target="$HOME" vscode
```

> **If stow errors with "existing target"**: The file already exists at the destination. Back it up and remove it first:
> ```bash
> mv ~/.zshrc ~/.zshrc.bak   # example for .zshrc
> stow --target="$HOME" zsh
> ```

Verify key symlinks:

```bash
ls -la ~/.zshrc ~/.gitconfig ~/.config/ghostty ~/.config/nvim ~/.config/yazi
# All should show -> pointing into ~/dotfiles/
```

---

## Step 8 — Reload shell

```bash
source ~/.zshrc
```

Or open a new terminal window. You should see:
- fastfetch system info on launch
- Oh My Posh prompt (Catppuccin colors, git branch info)
- Aliases available: `ll`, `ld`, `lf`, `mi`, `srch`

If the prompt shows garbled characters: the JetBrains Mono Nerd Font isn't set in your terminal. In Ghostty, it's already configured via the dotfiles config.

---

## Step 9 — First Neovim launch (LazyVim plugin install)

```bash
nvim
```

LazyVim will automatically download and install all plugins on first launch. Wait for it to complete (watch the progress in the `:Lazy` window). Exit and reopen nvim to confirm everything loaded cleanly.

---

## Step 10 — VS Code extensions

Open VS Code and install these extensions (Cmd+Shift+X):

- **Catppuccin Mocha** theme: `catppuccin.catppuccin-vsc`
- **Material Icon Theme**: `PKief.material-icon-theme`
- **GitHub Copilot**: `GitHub.copilot`
- **Python**: `ms-python.python`
- **Pylance**: `ms-python.vscode-pylance`

Or install via CLI:

```bash
code --install-extension catppuccin.catppuccin-vsc
code --install-extension PKief.material-icon-theme
code --install-extension GitHub.copilot
code --install-extension ms-python.python
```

---

## Step 11 — npm globals

```bash
npm install -g @anthropic-ai/claude-code
npm install -g notebooklm-mcp
```

---

## Step 12 — Python tools (via uv/pip)

```bash
pip3 install fastmcp mcp notebooklm-cli yt-dlp rich
```

---

## Step 13 — Manual steps (cannot be automated)

These require logins or account-specific setup:

1. **SSH keys** — Generate a new key pair or copy from old machine:
   ```bash
   ssh-keygen -t ed25519 -C "sayak1111@gmail.com"
   # Add ~/.ssh/id_ed25519.pub to GitHub: https://github.com/settings/keys
   ```

2. **GitHub CLI auth**:
   ```bash
   gh auth login
   # Choose GitHub.com → HTTPS → browser
   ```

3. **Google Cloud SDK** (if needed):
   ```bash
   brew install --cask google-cloud-sdk
   gcloud auth login
   gcloud config set project YOUR_PROJECT_ID
   ```

4. **Ghostty as default terminal** — Set in System Settings → Desktop & Dock → Default web browser (look for terminal setting) or right-click a .sh file.

5. **Rust/Cargo** (if needed):
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

---

## Verification Checklist

Run these after completing all steps to confirm setup is correct:

```bash
# Shell
echo $SHELL          # /bin/zsh
echo $ZSH            # ~/.oh-my-zsh

# Tools
which nvim fzf eza bat fd rg yazi micro oh-my-posh zoxide fastfetch
# All should print paths under /opt/homebrew/bin/

# Symlinks
ls -la ~/.zshrc ~/.gitconfig ~/.config/ghostty ~/.config/nvim ~/.config/yazi ~/.config/ohmyposh
# All should be symlinks -> ~/dotfiles/...

# Oh My Posh prompt
oh-my-posh --version

# Aliases
type ll ld lf srch mi
# All should resolve

# Neovim
nvim --headless -c "checkhealth" -c "qa" 2>&1 | tail -20
```

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `stow: existing target` | `mv <target> <target>.bak` then re-run stow |
| Prompt shows `?` boxes | Set terminal font to JetBrains Mono Nerd Font |
| `brew bundle` fails | Run `brew update && brew doctor` first |
| LazyVim plugins don't load | Delete `~/.local/share/nvim` and relaunch nvim |
| `zsh-autocomplete` slows shell | Remove it from plugins in `.zshrc` |
| Oh My Posh theme not found | Verify `~/.config/ohmyposh/cat.json` symlink exists |
