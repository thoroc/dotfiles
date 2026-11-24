# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A [chezmoi](https://www.chezmoi.io/) source directory for a macOS (web-dev focused) machine. Nothing here runs directly;
files are templates/sources that chezmoi renders into `$HOME`.

The source dir is `~/.dotfiles` (set via `sourceDir` in `.chezmoi.toml.tmpl` and the generated
`~/.config/chezmoi/chezmoi.toml`). The default chezmoi path `~/.local/share/chezmoi` is not used (the old checkout was
deleted); on a fresh machine, pass `--source ~/.dotfiles` to `chezmoi init` until the generated config exists.

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
- **`dot_claude/`** manages the portable parts of `~/.claude`, which must be a real directory (the standard location),
  not a symlink: chezmoi would replace a symlink with a directory. It covers (`CLAUDE.md`, `hooks/`,
  `settings.json.tmpl`, `dot_mcp.json.tmpl`, `obsidian/*.py.tmpl`). Skills, agents, plugins and runtime data are
  deliberately not managed and are excluded in `.chezmoiignore`; add new exclusions there, not by omission. Machine
  paths must use `{{ .chezmoi.homeDir }}`, never a literal home path, and secrets never go in these files (use `fnox`).
  Executable scripts need the `executable_` prefix. Everything Obsidian-related (MCP entry, two hooks, `obsidian/`) is
  gated on `.tools.notes.obsidian`, both inside the templates and in a templated block of `.chezmoiignore`. Read toggles
  with `dig "tools" "notes" "obsidian" false .` so templates still render on machines whose config predates the key.
- **Init defaults** live in `~/.config/chezmoi/init-defaults.toml` (per machine, untracked; template in
  `init-defaults.example.toml`, repo-only). `.chezmoi.toml.tmpl` reads it itself (`include`/`fromToml`, merged over the
  data as `$d`, which every `prompt*Once` call uses), so listed keys are not prompted and override stored answers. Do
  not rely on `chezmoi init --override-data-file`: it is ignored for prompts (only `execute-template` honours it). Keep
  emails, vault paths and other private values in that file, not in the repo.
- **Already-installed apps are not asked about.** For the container runtime and LLM runners the template uses a stored
  or defaults-file answer first, then treats an existing `/Applications/<App>.app` as yes, and only then prompts
  (`promptChoice`/`promptMultichoice`, not `...Once`, because the answer is derived). Reuse that pattern for other app
  prompts. Each `prompt*Once` path must match the key the data table writes (the Python prompt used `lang.python.enable`
  while the data key is `enabled`, so it re-prompted on every init).
- **Languages** are detected, not asked about, when already installed. `.chezmoi.toml.tmpl` resolves each binary with
  `lookPath`, classifies the path (`mise`, `brew`, `rustup`, `other`; `/usr/bin` stubs are ignored) and writes
  `lang.<name>.{enabled,source,migrate,manager}`. `manager` (`mise`, `brew`, `rustup`, `other`, `none`) is what
  consumers read: `dot_config/mise/config.toml.tmpl` only lists a language when its manager is `mise`, and the brew Java
  block only when it is `brew`. `migrate` is an opt-in prompt for Homebrew-sourced languages; it drives
  `run_onchange_after_darwin-migrate-brew-to-mise`, which runs `mise install` then a non-forcing `brew uninstall`
  (formulae with dependents stay, e.g. `python@3.x` and `openjdk` on most machines). Keep helper prompts (poetry, pyenv,
  jenv) on the same detect-first rule.
- **Brew tools moved to mise** are the multi-select `tools.migrate_to_mise` (plain names; mise and Homebrew use the same
  one). Candidates are an allowlist (`$movable` in `.chezmoi.toml.tmpl`) intersected with `brew list --formula`, so
  nothing is offered that is not installed. Only add a tool after proving it installs on both architectures from a
  prebuilt binary (use an isolated `MISE_DATA_DIR` and an empty `MISE_GLOBAL_CONFIG_FILE`); mise tools backed by
  `conda:` or an asdf plugin build or repackage software and are excluded. Chosen tools get a mise config entry and are
  handled by the same migration script as languages (`mise install`, then a non-forcing `brew uninstall`). Like the
  language prompts, the answer can be pre-set with `tools.migrate_to_mise = [...]` in `init-defaults.toml`.
- **Colima and local LLM runners.** The container runtime is one choice, `tools.container` (`docker` | `orbstack` |
  `colima` | `none`): exactly one, never several. The booleans under `tools.install.container.*` are derived from it for
  the existing consumers (casks, PATH, aliases). Detection finds Docker.app, OrbStack.app or a `colima` on PATH; one hit
  is used silently, several hits ask which, none asks with `docker` as the default. Colima is installed with a guarded
  `brew install colima` and is not in the move-to-mise allowlist, because who runs containers is the user's choice. LLM
  runners are the multi-select `tools.llm` (`ollama`, `jan`, `msty`, plus `lm-studio` on arm64 only), default none; apps
  already in `/Applications` are enabled without asking and the install script skips them
  (`[ -d app ] || brew install --cask`). Verify a cask's arch requirement
  (`brew info --json=v2 --cask X | jq '.casks[0].depends_on'`) before offering it on Intel.
- **Prompt and ls replacement** are single choices (`tools.prompt`: `oh-my-posh` | `none`, default `oh-my-posh`;
  `tools.ls`: `eza` | `lsd` | `none`, default `eza`). They gate the mise entries (`dot_config/mise/config.toml.tmpl`),
  the zprofile init block, the aliases, and the `.config/oh-my-posh` and `.config/lsd` applies (`.chezmoiignore`).
  Templates read them with `dig` and defaults equal to the previous behaviour, so older configs keep working. The old
  `tools.enable.lsd`/`eza`/`dasel` toggles (dasel was never consumed) and the unused `tools.list`, `tools.shell` and
  `tools.editor` keys are gone; stale values in an existing `chezmoi.toml` are harmless.
- **Terminal emulator** is one `promptChoiceOnce` (`tools.terminal`, default `iterm2`) whose value is also the Homebrew
  cask name, installed by `run_onchange_darwin-install-packages`; `terminal` (Terminal.app) installs nothing. Adding a
  terminal means adding it to the choice list in `.chezmoi.toml.tmpl` after confirming the cask name. The iTerm2 `it2`
  agent hooks in `dot_claude/settings.json.tmpl` are only emitted when it is `iterm2` (or unset, for older configs);
  events that contain only that hook are omitted entirely. The primary browser is a second choice, `tools.browser`
  (`safari` | `brave` | `firefox` | `chrome` | `edge`, default `safari`), independent of the install toggles.
  `run_onchange_after_darwin-configure-dock` (after the install script) pins the terminal, that browser and Apps.app in
  the Dock (Finder is always present; Apple's default apps are not pinned).
- **Editors/IDEs** are a multi-select stored as the list `tools.editors` (cask names, default `visual-studio-code`
  only), installed by looping over it in `run_onchange_darwin-install-packages`. `promptMultichoiceOnce` ignores stored
  and overridden lists (it would re-prompt on every `init` and ignore `init-defaults.toml`), so `.chezmoi.toml.tmpl`
  checks for an existing list with `dig`/`kindIs "slice"` and only then calls `promptMultichoice`. Templates read it
  with `dig "tools" "editors" (list "visual-studio-code") .` so older configs keep working. To test the prompt use
  `--promptMultichoice "Editors/IDEs to install=zed/cursor"` (values are `/`-separated).
- **AI coding assistants** are four independent toggles in `.chezmoi.toml.tmpl`
  (`tools.ai.claude|gemini|codex|opencode`, default false). Gemini and Codex are installed via mise
  (`dot_config/mise/config.toml.tmpl` gates each npm entry on its toggle). Claude Code and opencode ship their own
  installers and are usually already present, so the install script only runs `brew install` for them when `command -v`
  finds nothing. Never add an install for something already provided another way. `tools.enable.delta` (default true)
  gates delta: the mise entries, the Rust toolchain it needs on Intel, and the `[pager]`, `[interactive]` and `[delta]`
  sections of the gitconfig. On Intel, delta is compiled by mise (`"cargo:git-delta"`, pinned to a version so it only
  recompiles when bumped by hand) because there is no darwin/amd64 prebuilt; `rust` is added through mise when no other
  manager provides rustc. `dot_claude/` is applied only when `tools.ai.claude` is true (a templated `.claude` ignore in
  `.chezmoiignore`). Gemini, Codex and opencode have no managed config yet; add it under `dot_gemini/`, `dot_codex/` or
  `dot_config/opencode/` and gate it the same way. Existing machines must run `mise run init` to answer the new prompts:
  until then the `dig` defaults treat every assistant as off.
- **Notes tools** are three independent toggles in `.chezmoi.toml.tmpl` (`tools.notes.obsidian|notion|logseq`).
  `obsidian_vault` and `logseq_graph` paths are only prompted when the matching toggle is on. Only Obsidian is consumed
  by templates so far; Notion and Logseq are recorded but unused. To test prompts non-interactively:
  `chezmoi execute-template --init --promptBool "<prompt text>=true" < .chezmoi.toml.tmpl` (flags match the prompt text,
  not the key).
- **`Applist.txt`** is a snapshot of installed apps (repo-only, listed in `.chezmoiignore`).
- **`.chezmoiignore`** excludes repo-only files (README, `CLAUDE.md`, `hk.pkl`, `mise.toml`, `.prettierrc`,
  `.rumdl.toml`, `.vscode`, `Brewfile`) from being applied to `$HOME`. Add new repo-only files there, or chezmoi will
  copy them into `$HOME`.

## Conventions

- Markdown is wrapped by prettier (`--prose-wrap always`); keep the README TOC inside the doctoc markers.
- Commit messages follow conventional commits (`feat(scope):`, `fix(scope):`, `chore:`).
