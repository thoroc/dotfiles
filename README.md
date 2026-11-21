# Chezmoi dotfiles

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Prerequisite](#prerequisite)
- [Setup](#setup)
- [What gets applied](#what-gets-applied)
- [Day-to-day](#day-to-day)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

This repository uses [chezmoi](https://www.chezmoi.io/)

**target**: MacOS with webdev emphasis. Linux/Windows are not supported: the install and configure scripts only run on
darwin.

Other repositories using Chezmoi:

- <https://github.com/politician/dotfiles/>
- <https://github.com/federicober/dotfiles>
- <https://github.com/m0lson84/dotfiles>
- <https://github.com/pgumeson-fabric/dotfiles>
- <https://github.com/songkg7/dotfiles>
- <https://macos-defaults.com/>

## Prerequisite

install the following if not already present on the machine

- xcode: `xcode-select --install`
- git

Homebrew is installed automatically by the configure script if missing.

## Setup

Run:

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply thoroc
```

`init` prompts for identity (email, username, GitHub details), project directories, shell, and which optional tools,
languages and browsers to install. Answers are stored in `~/.config/chezmoi/chezmoi.toml`, and each question is only
asked once. The config also pins chezmoi's source directory to `~/.dotfiles`, so clone this repository there if you are
not using `init --apply thoroc` (which clones it to chezmoi's default location).

## What gets applied

- Shell config (zsh, aliases, oh-my-posh theme), git config, `lsd`, `fnox`, `mise` and vim config
- `.Brewfile` (linked to `~/.config/homebrew/.Brewfile`) and a toggle-driven `brew bundle` on every change
- macOS defaults (dock, Finder, screenshots, software update, TouchID for sudo) via `run_onchange` scripts
- A OneDrive LaunchAgent plist
- Tools listed in `dot_config/mise/config.toml`, managed by `mise` (upgrades wait 7 days after release)

## Day-to-day

Tasks are defined in `mise.toml`:

```sh
mise run plan     # chezmoi diff
mise run apply    # chezmoi apply
mise run init     # re-run chezmoi init
mise run format   # prettier on markdown
mise run lint     # rumdl on markdown
```

Git hooks are managed by [hk](https://hk.jdx.dev/) (`hk.pkl`) and run prettier, rumdl and doctoc on markdown files.
