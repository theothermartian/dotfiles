# New Mac Setup Guide

> **For LLMs:** This file is a complete, ordered instruction set to reproduce this Mac setup on a fresh machine. Execute every step in sequence. Each section includes verification commands. If a step fails, diagnose before continuing; do not skip.
>
> Before changing anything:
> - Inspect the current machine state first. Check whether Homebrew, Ghostty, `gh`, and the target config files already exist.
> - Do not append Homebrew init lines to `~/.zprofile` if this repo will later stow its own `.zprofile`.
> - Audit the repo for old machine-specific absolute paths before applying it:
>   ```bash
>   rg '/Users/|thereisnocake|Downloads/google-cloud-sdk' ~/dotfiles
>   ```
>   Replace old home-directory paths with `$HOME` where appropriate before stowing.
> - Prefer idempotent, non-interactive commands. Avoid installer scripts that rewrite shell startup files when a plain `git clone` will do.

---

## Prerequisites

- macOS 13+ (Ventura or later), Apple Silicon preferred
- Internet connection
- Apple ID signed in if App Store apps are needed
- This guide assumes a fresh or near-fresh user account

---

## Step 1 - Xcode Command Line Tools

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

## Step 2 - Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After install, load Homebrew into the current shell:

```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
```

> Do **not** append this to `~/.zprofile` here. The repo already tracks `zsh/.zprofile`, and writing to `~/.zprofile` now creates an avoidable stow conflict later.

Verify:

```bash
brew --version
```

---

## Step 3 - Clone dotfiles repo

```bash
git clone https://github.com/theothermartian/dotfiles ~/dotfiles
cd ~/dotfiles
```

Verify:

```bash
ls ~/dotfiles
```

---

## Step 4 - Install packages from Brewfile

```bash
brew bundle --file=~/dotfiles/Brewfile
```

This installs the CLI tools, fonts, terminal apps, editors, and most of the package-managed base.

> If `docker-desktop` fails with an error about `/usr/local/cli-plugins`, fix the directory and retry just that cask:
> ```bash
> sudo mkdir -p /usr/local/cli-plugins
> sudo chown "$USER":admin /usr/local/cli-plugins
> brew install --cask docker-desktop
> ```

Verify a few key tools:

```bash
nvim --version | head -1
fzf --version
eza --version
ghostty --version
```

---

## Step 5 - Install Oh My Zsh

```bash
git clone https://github.com/ohmyzsh/ohmyzsh.git ~/.oh-my-zsh
```

This installs to `~/.oh-my-zsh` without running the interactive installer or rewriting shell startup files.

Verify:

```bash
ls ~/.oh-my-zsh
```

---

## Step 6 - Install zsh plugins

The `.zshrc` uses three plugins that are not bundled with Oh My Zsh:

```bash
ZSH_CUSTOM="${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}"
mkdir -p "$ZSH_CUSTOM/plugins"

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
```

---

## Step 7 - Apply dotfiles with GNU stow

First dry-run to detect conflicts:

```bash
cd ~/dotfiles
stow -nv --target="$HOME" zsh github ghostty ohmyposh micro nvim yazi zed vscode
```

If any targets already exist, remove or back them up before continuing. Common examples:

```bash
rm -f ~/.zprofile ~/.zshrc ~/.gitconfig
rm -rf ~/.config/ghostty ~/.config/ohmyposh ~/.config/micro ~/.config/nvim ~/.config/yazi ~/.config/zed
rm -rf "$HOME/Library/Application Support/Code"
```

Then apply the packages:

```bash
cd ~/dotfiles
stow --target="$HOME" zsh github ghostty ohmyposh micro nvim yazi zed vscode
```

Verify key symlinks:

```bash
ls -la ~/.zshrc ~/.zprofile ~/.gitconfig ~/.config/ghostty ~/.config/nvim ~/.config/yazi
```

---

## Step 8 - Sync Yazi plugins

The Yazi config expects external plugins declared in `package.toml`:

```bash
ya pkg install
```

Verify:

```bash
find ~/.local/state/yazi/packages -maxdepth 2 -type d | head
```

> Note: `ya pkg install` may rewrite `package.toml`. If the repo carries custom previewer entries there, verify they still exist after the install.

---

## Step 9 - Reload shell

```bash
exec zsh -l
```

Or open a new terminal window. You should see:
- `fastfetch` system info on launch
- Oh My Posh prompt
- Aliases available: `ll`, `ld`, `lf`, `mi`, `srch`

If the prompt shows garbled characters, make sure the terminal font is set to JetBrains Mono Nerd Font. Ghostty is already configured for this repo.

---

## Step 10 - First Neovim launch

```bash
nvim
```

LazyVim will automatically download and install all plugins on first launch. Wait for it to complete, then exit and reopen `nvim` to confirm everything loaded cleanly.

> If automating headlessly, use:
> ```bash
> nvim --headless "+Lazy! restore" +qa
> ```
> Then still launch `nvim` once interactively. Some Mason installs can still be finishing during the first headless bootstrap.

---

## Step 11 - VS Code extensions

If the `code` CLI is not in `PATH`, open VS Code once and run:

- `Shell Command: Install 'code' command in PATH`

Then install these extensions:

- `catppuccin.catppuccin-vsc`
- `PKief.material-icon-theme`
- `GitHub.copilot`
- `ms-python.python`
- `ms-python.vscode-pylance`

CLI form:

```bash
code --install-extension catppuccin.catppuccin-vsc
code --install-extension PKief.material-icon-theme
code --install-extension GitHub.copilot
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
```

---

## Step 12 - npm globals

```bash
npm install -g @anthropic-ai/claude-code
npm install -g notebooklm-mcp
```

---

## Step 13 - Python tools

```bash
pip3 install fastmcp mcp notebooklm-cli yt-dlp rich
```

---

## Step 14 - Manual steps

These require logins or account-specific setup:

1. **SSH keys**
   ```bash
   ssh-keygen -t ed25519 -C "sayak1111@gmail.com"
   ```
   Add `~/.ssh/id_ed25519.pub` to GitHub.

2. **GitHub CLI auth**
   ```bash
   gh auth login
   ```

3. **Google Cloud SDK** if needed
   ```bash
   brew install --cask google-cloud-sdk
   gcloud auth login
   gcloud config set project YOUR_PROJECT_ID
   ```

4. **Docker Desktop first launch**
   Open Docker from `/Applications` once and let it finish first-run setup.

5. **Ghostty as primary terminal**
   Open Ghostty manually and use it as the main terminal. macOS does not provide a single system-wide "default terminal" setting like it does for browsers.

6. **Rust/Cargo** if needed
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

---

## Verification Checklist

Run these after completing all steps:

```bash
# Shell
echo $SHELL
echo $ZSH

# Tools
which nvim fzf eza bat fd rg yazi micro oh-my-posh zoxide fastfetch

# Symlinks
ls -la ~/.zshrc ~/.zprofile ~/.gitconfig ~/.config/ghostty ~/.config/nvim ~/.config/yazi ~/.config/ohmyposh

# Oh My Posh
oh-my-posh --version

# Aliases
type ll ld lf srch mi

# Yazi packages
find ~/.local/state/yazi/packages -maxdepth 2 -type d | head

# Docker
test -d /Applications/Docker.app && echo "Docker OK"

# Neovim
nvim --headless -c "checkhealth" -c "qa" 2>&1 | tail -20
```

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `stow: existing target` | Remove or back up the conflicting target, then re-run `stow` |
| Prompt shows `?` boxes | Set terminal font to JetBrains Mono Nerd Font |
| `brew bundle` fails on `docker-desktop` | Create `/usr/local/cli-plugins`, fix ownership, then retry the cask |
| LazyVim plugins don't load | Delete `~/.local/share/nvim` and relaunch `nvim` |
| Mason/treesitter installs race in headless bootstrap | Run the headless bootstrap once, then open `nvim` interactively and let installs finish |
| `zsh-autocomplete` slows shell | Remove it from `plugins=(...)` in `.zshrc` |
| Oh My Posh theme not found | Verify `~/.config/ohmyposh/cat.json` symlink exists |
| Yazi plugins missing | Re-run `ya pkg install` after the dotfiles are stowed |
