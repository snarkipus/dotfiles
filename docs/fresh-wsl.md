# Fresh Ubuntu WSL setup

These steps are manual and reviewable. Do not run them as one unattended script.
Keep a working Bash terminal open until the new shell has been tested.

## 1. Prepare WSL and the terminal

If Ubuntu WSL is not installed, use your approved Windows software-management
procedure. On an unmanaged personal Windows system, an elevated PowerShell can
list distributions with `wsl --list --online` and install Ubuntu with
`wsl --install -d Ubuntu`. Restart if requested and complete Ubuntu's user setup.
Do not change organization-managed Windows features without approval.

Install a Nerd Font in Windows if permitted, then select its exact family name
under Windows Terminal > Settings > Ubuntu > Appearance > Font face. The Mono
variant is a sensible choice. If it is missing from the picker, check Windows
Settings > Personalization > Fonts, enable Show all fonts in Terminal if
available, and restart Terminal. An icon-free configuration is also possible.

## 2. Install tools through approved repositories

Inside Ubuntu:

```sh
. /etc/os-release
printf '%s\n' "$PRETTY_NAME"
uname -m
apt-cache policy zsh starship tmux
```

If these packages are available from approved repositories:

```sh
sudo apt update
sudo apt install zsh starship tmux
zsh --version
starship --version
tmux -V
```

Review the proposed package transaction. This is not a distribution upgrade.
If a package has no candidate, stop and choose an approved source rather than
adding arbitrary repositories or piping a remote installer into a shell.

The initial configuration target is Zsh 5.9, Starship 1.22.1, and tmux 3.6.
Do not assume every Ubuntu release provides those versions. Preserve Bash as
the default shell during testing. Neovim and Mise are not prerequisites.

## 3. Download and verify an actual release asset

Download the explicitly attached configuration archive and its SHA-256 file
from this repository's Releases page through an allowed browser/transfer route.
GitHub's automatic Source code archives are not substitutes for the bundle.
There is no requirement to clone GitHub from WSL.

Place the files in a temporary working directory. For the initial asset names:

```sh
sha256sum -c company-shell.tar.gz.sha256
tar -tzf company-shell.tar.gz
```

Require a matching checksum. A checksum detects transfer changes; it does not
prove publisher trust or constitute a malware scan. Inspect the file list: it
should contain only the documented `home/` tree, no absolute paths, `..` traversal,
credentials, history, or personal runtime state. Reject escaping links and
unexpected file types. Do not extract an untrusted archive just because its
checksum matches an accompanying file.

For a trusted, reviewed release, extract without applying archive ownership:

```sh
stage=$(mktemp -d "$HOME/dotfiles-test.XXXXXXXX")
chmod 700 "$stage"
tar --no-same-owner -xzf company-shell.tar.gz -C "$stage"
```

The initial staged tmux-resurrect payload has three known dangling test-harness
links to an omitted upstream test submodule. They are not runtime dependencies.
Any other dangling or escaping link needs review, not a blanket exception.

## 4. Test the prompt and parse the shell

```sh
zsh -dfn "$stage/home/.zshrc"
(
  cd "$stage"
  STARSHIP_CONFIG="$stage/home/.config/starship.toml" starship prompt
)
```

Require the intended prompt and no configuration warnings. Language/project
segments appear only in relevant directories. This standalone test does not
prove every conditional module or the right prompt.

A preview using the staged `.zshrc`, without changing the default shell:

```sh
ZDOTDIR="$stage/home" \
STARSHIP_CONFIG="$stage/home/.config/starship.toml" zsh -i
```

This uses your real HOME: it can create Zsh completion cache/history, and it
loads plugins from the real HOME paths if already installed. It is not a fully
isolated test and does not yet test the staged plugin trees. Test ordinary
commands, Up Arrow, and Tab; use `exit` to return to Bash.

## 5. Install after review, with a backup

Review the staged files and plugin licenses. The intended destination set is
only `.zshrc`, `.tmux.conf`, `.config/starship.toml`, and the eight named plugin
directories listed in the README.

Before overwriting anything, preserve existing files and plugin directories in
a private backup outside those destinations and record which targets were absent.
Do not back up or replace all of `.config` or `.local/share` for this operation.
If a destination is an unexpected symlink, stop and inspect it first.

Copy the three configuration files to the corresponding HOME paths and each
named plugin directory to its matching `.local/share/.../plugins/` path. Preserve
plugin executable bits and contained symlinks. Use ordinary files at mode 0644,
plugin executables at 0755, and non-group-writable directories. Do not copy the
export's `originals/` directory, caches, `.git`, or session saves. Do not use
`rsync --delete` against HOME. This guide intentionally does not provide a broad
one-command overwrite of an existing home directory.

For a truly fresh destination, the three config files can be installed with:

```sh
mkdir -p "$HOME/.config"
install -m 644 "$stage/home/.zshrc" "$HOME/.zshrc"
install -m 644 "$stage/home/.tmux.conf" "$HOME/.tmux.conf"
install -m 644 "$stage/home/.config/starship.toml" "$HOME/.config/starship.toml"
```

These commands overwrite existing files: only use them after the backup/review
above.

**The plugin copy is required for the theme and Zsh plugins.** Run this block in
Bash, in the same terminal where `$stage` was set. It checks every source and
refuses to overwrite existing plugin directories before copying anything:

```sh
(
  set -eu
  : "${stage:?Set stage to the extracted bundle directory first}"
  plugins=(
    zsh/zsh-autosuggestions
    zsh/zsh-syntax-highlighting
    tmux/catppuccin
    tmux/tmux-sensible
    tmux/tmux-cpu
    tmux/vim-tmux-navigator
    tmux/tmux-resurrect
    tmux/tmux-continuum
  )
  for item in "${plugins[@]}"; do
    relative=".local/share/${item%%/*}/plugins/${item#*/}"
    source="$stage/home/$relative"
    destination="$HOME/$relative"
    if [ ! -d "$source" ] || [ -L "$source" ]; then
      printf 'STOP: missing or symlinked source: %s\n' "$source" >&2
      exit 1
    fi
    if [ -e "$destination" ] || [ -L "$destination" ]; then
      printf 'STOP: existing destination needs review: %s\n' "$destination" >&2
      exit 1
    fi
  done
  for item in "${plugins[@]}"; do
    relative=".local/share/${item%%/*}/plugins/${item#*/}"
    mkdir -p "$HOME/$(dirname "$relative")"
    cp -a -- "$stage/home/$relative" "$HOME/$relative"
  done
  test -f "$HOME/.tmux.conf"
  test -f "$HOME/.local/share/tmux/plugins/catppuccin/catppuccin.tmux"
  printf 'CONFIG_AND_PLUGINS_INSTALLED\n'
)
```

The archive already contains these plugins; no plugin download is necessary.
If a source is missing, verify you extracted the release asset rather than the
automatic GitHub Source code archive. If `$stage` was lost after closing the
terminal, set it to the actual extraction directory, not its `home/` child.

## 6. Verify the installed shell and tmux

Start `zsh` from the retained Bash terminal. Check the prompt, aliases, completion,
keybindings, and plugin behavior. Missing optional binaries do not get installed
automatically. Use `exit` to return to Bash if anything is wrong.

Test tmux with a dedicated server rather than reloading an existing session:

```sh
tmux -L dotfiles-check -f "$HOME/.tmux.conf" new-session -s check
```

Ensure that socket name is not already in use for valuable sessions. This server
is separate from the normal tmux server, but plugins still use your real HOME
and can write runtime state or restore locally saved sessions. A fresh machine
should not have imported session saves. Check for startup errors, status modules,
pane splits, clipboard behavior, and terminal key handling. The configured
prefix is Ctrl+A, not Ctrl+B. Detach with Ctrl+A then D, and terminate only the
test server:

```sh
tmux -L dotfiles-check kill-server
```

Do not call `tmux kill-server` without the test socket selector. Terminal and WSL
clipboard/key behavior may vary independently of configuration syntax.

## 7. Optionally select Zsh as your login shell

Only after successful interactive testing:

```sh
command -v zsh
chsh -s "$(command -v zsh)"
```

The selected path must be listed in `/etc/shells`, and policy must permit the
change. Open a new Ubuntu terminal to verify; keep the old Bash session open.
A Windows Terminal profile explicitly launching Bash may need its launch command
reviewed separately. Do not blindly replace WSL or terminal settings.

## Recovery and updates

Return to Bash and restore the backed-up exact destinations. Remove only targets
recorded as absent before installation; never recursively delete broad HOME
folders. Revert the login shell to its recorded original value if changed.

For an update, download a new version, verify it, compare with the installed
snapshot, back up the affected paths, and repeat the relevant tests. Runtime
history, cache, and saved sessions should remain local. No automatic updater or
GitHub access at shell startup is required for the bundled plugins.
