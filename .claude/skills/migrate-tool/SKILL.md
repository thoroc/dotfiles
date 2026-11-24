---
name: migrate-tool
description:
  Move a tool that is configured locally on this machine into the dotfiles repo (chezmoi source), with a per-platform
  install path. Use when asked to "add", "migrate", "manage" or "bring <tool> into the dotfiles".
---

<!-- DOCTOC SKIP -->

# Migrate a locally configured tool into the dotfiles

Conventions live in `CLAUDE.md` (repo root). Read its Architecture and Conventions sections first; do not restate them
here. Platforms: macOS Intel, macOS arm, Linux (Debian/Ubuntu). Work one step at a time and say what you checked (blast
radius).

## 1. Detect how it is installed here

`type -a <tool>`, `mise ls | rg <tool>`, `brew list --formula --cask | rg <tool>`, `/Applications`,
`apt list --installed`. Watch for duplicate or stale binaries. Find its config: `~/.config/<tool>`, `~/.<tool>rc`,
`~/Library/Application Support`.

## 2. Triage what is in the config

| Kind           | Examples                                                           | Action                                                                 |
| -------------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| Config         | settings, themes, aliases, keymaps                                 | Migrate                                                                |
| State          | caches, history, sessions, databases, lockfiles, plugin dirs       | Do not migrate; add to `.chezmoiignore` if the dir is shared           |
| Secrets        | tokens, API keys, passwords, private keys                          | Never commit; route to `fnox`                                          |
| Self-rewriting | files the tool rewrites at runtime (window geometry, recent files) | Flag to the user; prefer migrating a stable subset or a symlink target |

Scan before copying: `rg -i "token|secret|password|api[_-]?key|BEGIN .*PRIVATE|://[^/ ]*:[^@ ]*@"` on the candidate
files. Anything found becomes an `fnox` entry (keychain on macOS, age on Linux), with the config reading it from the
environment.

## 3. Decide the install path per platform

Prefer, in order: mise (prebuilt binary, `github:` or registry backend) > brew formula/cask (macOS only) > apt (Linux
system packages) > the tool's own installer (guarded with `command -v`). Never add an install for something already
provided another way.

| Platform    | Where it goes                                                                                                                 |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------- |
| all         | `dot_config/mise/config.toml.tmpl` (only if a prebuilt binary exists on every arch/OS; test with an isolated `MISE_DATA_DIR`) |
| macOS only  | inline bundle in `.chezmoiscripts/run_onchange_darwin-install-packages.sh.tmpl` (and `Brewfile`)                              |
| Linux only  | `.chezmoiscripts/run_onchange_after_linux-install-packages.sh.tmpl` (apt list)                                                |
| desktop app | macOS cask only; gate Linux off                                                                                               |

If the tool should be optional, add a toggle: prompt in `.chezmoi.toml.tmpl` (macOS-only prompts inside the `$darwin`
guard), consume with `dig` and a default equal to the old behaviour, and document it in `init-defaults.example.toml`.

## 4. Edit the source

- Config goes under `dot_config/<tool>/` (chezmoi naming: `dot_`, `private_`, `executable_`, `symlink_`). Use a `.tmpl`
  only when it needs data: machine paths use `{{ .chezmoi.homeDir }}`, never a literal home path.
- Gate OS-specific content on `.chezmoi.os`; if the tool is optional, also add an ignore block in `.chezmoiignore`.
- Shell integration: shared env goes in `dot_config/zsh/dot_common.zsh.tmpl`, interactive init in `dot_zshrc.tmpl`,
  login init in `dot_zprofile.tmpl`, aliases in `dot_aliases.zsh.tmpl`. Guard with `command -v`.
- Update `CLAUDE.md` (and `README.md` if it lists the tool) when a new toggle or convention appears.

## 5. Verify

1. `chezmoi execute-template --source ~/.dotfiles < <file>` for every touched template, then parse the output (`zsh -n`,
   a TOML/JSON parser).
2. `mise run plan` (`chezmoi diff`): only the intended files change.
3. Linux: `execute-template` cannot fake `.chezmoi.os`; run `chezmoi init` + `apply` in a Debian container
   (`CHEZMOI_INIT_DEFAULTS` pointing at a full answers file, mount the repo read-only, no `rm` in the script).
4. Exercise both branches: toggle on and off, tool present and missing.
5. Commit only the files that belong to the change (`git commit -o <files>`), conventional message.
