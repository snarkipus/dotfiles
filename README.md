# dotfiles

Portable Zsh, Starship, and tmux configuration snapshots for Ubuntu on WSL.

Keep the familiar prompt, keybindings, and shell behavior without importing an
entire personal workstation or requiring plugin downloads at startup.

## Availability

**Repository and instructions are published; the initial configuration bundle is
not attached yet.** A draft release is reserved for the first upload. GitHub's
automatic source-code ZIP/tar downloads currently contain these documents, not
the configuration bundle. Use an explicitly attached release asset when available.

- [Fresh WSL setup](docs/fresh-wsl.md)
- [Publishing a configuration snapshot](docs/publishing.md)
- [Releases](https://github.com/snarkipus/dotfiles/releases)

## Intended bundle

```text
home/
├── .zshrc
├── .tmux.conf
├── .config/starship.toml
└── .local/share/
    ├── zsh/plugins/
    └── tmux/plugins/
```

The selected plugin trees are zsh-autosuggestions, zsh-syntax-highlighting,
Catppuccin tmux, tmux-sensible, tmux-cpu, vim-tmux-navigator, tmux-resurrect, and
tmux-continuum. Their upstream licenses must accompany their source.

The bundle excludes credentials, personal Git identity, Git history, shell
history, saved tmux sessions, Neovim, package installations, generated caches,
and workstation-specific environment setup. It does not replace `.profile` or
`.zshenv`, change the default shell, or run Chezmoi lifecycle scripts.

This repository is a distribution surface for reviewed exports. It is not an
automatic mirror of a personal dotfiles repository. Chezmoi is not required on
the receiving machine.

## Compatibility status

The current export target is Zsh 5.9, Starship 1.22.1, and tmux 3.6 on Ubuntu WSL.
These are compatibility targets, not minimum-version guarantees.

- The adapted Zsh configuration passed a parser-only check under Zsh 5.9.
- The adapted Starship configuration rendered the intended prompt under 1.22.1
  without reported warnings or errors. Pixi configuration was removed because
  that version does not provide the module.
- Full interactive Zsh behavior and the complete tmux/plugin combination remain
  to be tested on the receiving installation.
- Optional commands are not supplied by the configuration. For example, missing
  `gitmux` means the guarded Git segment in tmux is absent.
- Nerd Font glyphs require a suitable font selected in the Windows terminal
  emulator, not just installed inside WSL.

A public download is not an organizational software approval. Use this material
only through transfer and installation routes permitted for the receiving system.

## Maintenance

Edit and test the authoritative source configuration, then export a new reviewed
snapshot. Publish versioned assets with checksums; do not silently replace an
already published asset. Local runtime caches and history stay local.

For Chezmoi-managed templates, the normal source loop is targeted `chezmoi edit`,
`chezmoi diff`, an explicitly scoped apply, a behavior check, and a Git commit.
Direct edits to live files need to be reconciled into the source before export.
Do not use a broad `chezmoi update` as an export or installation command.

## Licensing

No blanket license is assigned to future third-party bundle content. Preserve
upstream license notices and record plugin provenance when publishing. Until an
explicit license is added for original material, no additional license grant is
implied by the repository being public.
