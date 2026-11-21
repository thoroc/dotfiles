# Chezmoi dotfiles

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Prerequisite](#prerequisite)
- [Setup](#setup)
- [What gets applied](#what-gets-applied)
- [Claude Code config](#claude-code-config)
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
git clone https://github.com/thoroc/.dotfiles ~/.dotfiles
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply --source ~/.dotfiles
```

`init` prompts for identity (email, username, GitHub details), project directories, shell, which notes tools you use,
and which optional tools, languages and browsers to install. Answers are stored in `~/.config/chezmoi/chezmoi.toml`, and
each question is only asked once. The config pins chezmoi's source directory to `~/.dotfiles`, so always clone there and
pass `--source ~/.dotfiles`: chezmoi's default location (`~/.local/share/chezmoi`) is not used.

To skip the prompts, copy `init-defaults.example.toml` to `~/.config/chezmoi/init-defaults.toml` and edit it (it stays
outside the repo, so private values such as vault paths are not committed). `chezmoi init` (and `mise run init`) reads
that file from the config template; keys in it are used instead of prompting and win over previously stored answers. Set
`CHEZMOI_INIT_DEFAULTS` to read a different file.

The shell prompt (`oh-my-posh` by default, or `none`) and the `ls` replacement (`eza` by default, `lsd` or `none`) are
single choices too. Only the selected tool is installed through `mise`, and only its config (the oh-my-posh theme, the
lsd config) is applied.

The terminal emulator is a single choice (`iterm2` by default, or `warp`, `alacritty`, `kitty`, `hyper`, `wezterm`,
`terminal`). The chosen one is installed as a Homebrew cask; `terminal` means the built-in Terminal.app and installs
nothing.

Editors and IDEs are a multi-select that defaults to Visual Studio Code only. Alternatives are Zed, Cursor, VSCodium,
Sublime Text, WebStorm, IntelliJ IDEA and PyCharm; each selected one is installed as a Homebrew cask. In
`init-defaults.toml` the key is `tools.editors`, a list of cask names.

AI coding assistants (Claude Code, Gemini CLI, Codex CLI, opencode) are independent yes/no answers that default to no.
Gemini CLI and Codex CLI are installed through `mise` (npm backend). Claude Code and opencode are installed with
Homebrew only when `claude` or `opencode` is not already on the PATH, so an existing install is left alone. Only Claude
Code has managed config so far (see below), and it is applied only when Claude Code is enabled.

Notes tools (Obsidian, Notion, Logseq) are independent yes/no answers, so any combination works, including none.
Obsidian additionally asks for a vault path and Logseq for a graph path; neither is asked unless that tool is selected.

## What gets applied

- Shell config (zsh, aliases, oh-my-posh theme), git config, `lsd`, `fnox`, `mise` and vim config
- `.Brewfile` (linked to `~/.config/homebrew/.Brewfile`) and a toggle-driven `brew bundle` on every change
- Optional container runtimes (Docker Desktop, OrbStack, both or neither); PATH and aliases are only generated for the
  ones you choose; an app that is already installed is enabled without asking
- macOS defaults (dock, Finder, screenshots, software update, TouchID for sudo) via `run_onchange` scripts
- A OneDrive LaunchAgent plist
- Tools listed in `dot_config/mise/config.toml`, managed by `mise` (upgrades wait 7 days after release)

## Claude Code config

`dot_claude/` manages the portable parts of `~/.claude` (only when Claude Code is enabled):

- `CLAUDE.md`, `commands/`, the `cbm-*` hook scripts
- `settings.json` and `.mcp.json`, rendered from templates so machine paths use the home directory instead of being
  hardcoded
- With iTerm2 as the terminal only: the `it2` agent-status hooks (all other hooks are unaffected)
- With Obsidian enabled only: the Obsidian MCP server entry, its two hooks (`obsidian-context.sh`, `capture-insight.sh`)
  and the `obsidian/*.py` scripts, with the vault path filled in from your answer

Not managed (see `.chezmoiignore`): skills, agents, plugins, and all runtime or session data (`projects/`,
`history.jsonl`, `cache/`, `sessions/`, `venvs/`, the Obsidian vector database, and so on). No secrets live in these
files; keep tokens in `fnox`. Changes are committed manually, with no auto-commit or auto-push hooks. Preview with
`mise run plan` before applying.

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
