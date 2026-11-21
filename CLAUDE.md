# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A [chezmoi](https://www.chezmoi.io/) source directory for a macOS (web-dev focused) machine. Nothing here runs directly;
files are templates/sources that chezmoi renders into `$HOME`.

The source dir is `~/.dotfiles` (set via `sourceDir` in `.chezmoi.toml.tmpl` and the generated
`~/.config/chezmoi/chezmoi.toml`). The default chezmoi path `~/.local/share/chezmoi` is a stale 2024 checkout; always
pass `--source ~/.dotfiles` if `chezmoi source-path` does not return `~/.dotfiles`.

## Commands

Tasks are defined in `mise.toml` (`mise run <task>`):

- `mise run plan` - `chezmoi diff`, preview changes (do this before apply)
- `mise run apply` - `chezmoi apply`
- `mise run init` - `chezmoi init` (regenerates config from `.chezmoi.toml.tmpl`; re-prompts for new keys only)
- `mise run format` / `mise run lint` - prettier / rumdl on markdown
- Render a single template to check syntax: `chezmoi execute-template --source ~/.dotfiles < path/to/file.tmpl`

Git hooks are managed by `hk` (`hk.pkl`, installed via `mise run prepare` and on entering the dir). They only cover
markdown: prettier (`--prose-wrap always`), rumdl, and doctoc for TOCs. There is no test suite.

## Architecture

- **`.chezmoi.toml.tmpl`** is the config source. It prompts once (`promptStringOnce`/`promptBoolOnce`) for identity,
  project dirs, shell, and feature toggles, and writes them into `[data]` in `~/.config/chezmoi/chezmoi.toml`. Every
  other template reads these as `.tools.install.*`, `.lang.*`, `.user.work/github.*`, `.os_shell`, `.homebrew_prefix`,
  etc. Adding a new toggle means adding a prompt here _and_ consuming it in templates. Because of `*Once`, existing
  machines keep old answers until the key is removed from their config.
- **chezmoi naming conventions** determine targets: `dot_` -> `.`, `private_` -> restricted permissions,
  `symlink_*.tmpl` -> a symlink whose target is the rendered content (e.g. `symlink_dot_zshrc.tmpl` links `~/.zshrc` to
  `~/.config/zsh/.zshrc`; `symlink_dot_Brewfile.tmpl` links to `~/.config/homebrew/.Brewfile`). Shell config therefore
  lives in `dot_config/zsh/`, not at the home root.
- **`.chezmoiscripts/`** hold `run_onchange_darwin-*` scripts that re-run whenever their rendered content changes:
  `install-packages` (conditional `brew bundle` driven by the toggles) and `configure-apps` (Homebrew bootstrap, macOS
  `defaults write`, sudo TouchID, dock, etc.). Both are wrapped in `{{ if eq .chezmoi.os "darwin" }}`. Template comment
  or brace errors here break `chezmoi apply` entirely.
- **`Brewfile`** at the repo root is the full static package list, separate from the toggle-driven inline bundle in
  `install-packages`. Custom-tap formulae have been migrating to the mise github backend (see git log).
- **`dot_config/`** holds tool configs: `zsh` (aliases, profile, env, rc), `gitconfig` (main plus github/gitlab
  includes, global gitignore), `mise`, `oh-my-posh`, `lsd`, `fnox`. `mise/config.toml` is the global tool list (the
  user's installed CLIs) and sets a 7-day upgrade cooloff; do not confuse it with the repo-local `mise.toml` (tasks).
- **`thomas.roche_Applist.txt`** is a snapshot of installed apps. It is not ignored, so chezmoi currently copies it into
  `$HOME`.
- **`.chezmoiignore`** excludes repo-only files (README, `CLAUDE.md`, `hk.pkl`, `mise.toml`, `.prettierrc`,
  `.rumdl.toml`, `.vscode`, `Brewfile`) from being applied to `$HOME`. Add new repo-only files there, or chezmoi will
  copy them into `$HOME`.

## Conventions

- Markdown is wrapped by prettier (`--prose-wrap always`); keep the README TOC inside the doctoc markers.
- Commit messages follow conventional commits (`feat(scope):`, `fix(scope):`, `chore:`).
