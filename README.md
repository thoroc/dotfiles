# Chezmoi dotfiles

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Prerequisite](#prerequisite)
- [Setup](#setup)
- [What gets applied](#what-gets-applied)
- [Claude Code config](#claude-code-config)
- [Day-to-day](#day-to-day)
- [GitHub tokens](#github-tokens)

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

Languages (Node, Python, Go, Rust, Java) that are already on the machine are enabled without asking. `init` records
where each one comes from (`mise`, `brew`, `rustup` or `other`; the `/usr/bin` stubs do not count) and only mise-managed
or missing languages get a mise entry, so nothing is installed twice. For a Homebrew language it offers to move it to
mise: if you accept, `mise install` runs first and the Homebrew formula is then removed unless other formulae depend on
it (that is reported, never forced). mise has no built-in migration; `mise sync node`/`ruby` can link Homebrew versions
of those two, but everything else is a fresh `mise install` followed by `brew uninstall`.

A few Homebrew tools that mise can provide as prebuilt binaries (currently `d2` and `mas`) can be moved the same way:
`init` lists the ones you have installed with Homebrew and you pick which to move (none by default). `ffmpeg`, `make`
and `poppler` are not offered, because mise would build or repackage them rather than ship a binary.

The container runtime is exactly one of Docker Desktop, OrbStack or Colima (or none), never several. If one is already
installed it is used without asking; if more than one is installed you are asked which to use; if none is, you pick
(Docker Desktop is the default). Local LLM runners (Ollama, Jan, Msty, and LM Studio on Apple Silicon) are a
multi-select that defaults to none; any that are already installed are enabled without asking and are never installed a
second time.

The terminal emulator is a single choice (`iterm2` by default, or `warp`, `alacritty`, `kitty`, `hyper`, `wezterm`,
`terminal`). The chosen one is installed as a Homebrew cask; `terminal` means the built-in Terminal.app and installs
nothing.

Editors and IDEs are a multi-select that defaults to Visual Studio Code only. Alternatives are Zed, Cursor, VSCodium,
Sublime Text, WebStorm, IntelliJ IDEA and PyCharm; each selected one is installed as a Homebrew cask. In
`init-defaults.toml` the key is `tools.editors`, a list of cask names.

AI coding assistants (Claude Code, Gemini CLI, Codex CLI, opencode) are independent yes/no answers that default to no.
Gemini CLI and Codex CLI are installed through `mise` (npm backend). Claude Code and opencode are installed only when
missing: the install script checks `~/.local/bin/claude` and `~/.opencode/bin/opencode` as well as the PATH (those
directories are not on the PATH inside chezmoi scripts), runs the official installer, and falls back to Homebrew with a
warning if that fails. On Intel the opencode formula has no bottle and Homebrew would compile llvm from source (hours),
so abort that fallback and install it by hand. Only Claude Code has managed config so far (see below), and it is applied
only when Claude Code is enabled.

Notes tools (Obsidian, Notion, Logseq) are independent yes/no answers, so any combination works, including none.
Obsidian additionally asks for a vault path and Logseq for a graph path; neither is asked unless that tool is selected.

To re-run setup on a machine that already has the config, use `mise run init` (only new keys are prompted), then
`mise run plan` and `mise run apply`. Read the plan first: `apply` overwrites any managed file that was edited in place,
for example `~/.claude/settings.json`. `run_onchange` scripts only re-run when their rendered content changes, so
`chezmoi apply --include=scripts` printing nothing means they are up to date; it does not apply config files. Apply a
single file with `chezmoi apply <target>`. The brew install step never runs `brew update` or upgrades casks, so upgrade
those yourself.

## What gets applied

- Shell config (zsh, aliases, oh-my-posh theme), git config, `lsd`, `fnox`, `mise` and vim config
- `.Brewfile` (linked to `~/.config/homebrew/.Brewfile`) and a toggle-driven `brew bundle` on every change
- One container runtime (Docker Desktop, OrbStack, Colima or none); PATH and aliases are only generated for the one you
  choose, and one that is already installed is used without asking
- macOS defaults (dock, Finder, screenshots, software update, TouchID for sudo) via `run_onchange` scripts
- A OneDrive LaunchAgent plist
- Tools listed in `dot_config/mise/config.toml`, managed by `mise` (upgrades wait 7 days after release)

## Claude Code config

`dot_claude/` manages the portable parts of `~/.claude` (only when Claude Code is enabled):

- `CLAUDE.md`, the `cbm-*` hook scripts
- `settings.json` and `.mcp.json`, rendered from templates so machine paths use the home directory instead of being
  hardcoded
- With iTerm2 as the terminal only: the `it2` agent-status hooks (all other hooks are unaffected)
- With Obsidian enabled only: the Obsidian MCP server entry, its two hooks (`obsidian-context.sh`, `capture-insight.sh`)
  and the `obsidian/*.py` scripts, with the vault path filled in from your answer
- The user-scope `codebase-memory-mcp` server: Claude Code does not load user-wide servers from `~/.claude/.mcp.json`,
  so `.chezmoiscripts/run_after_register-claude-mcp` registers it with `claude mcp add` whenever the entry in
  `~/.claude.json` is missing or points elsewhere (a custom command there is replaced). If `claude` or the server binary
  is not installed yet, it says so and retries on the next apply

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

## GitHub tokens

No long-lived GitHub token is exported to the shell. The classic PAT (no scopes) is exported only as
`HOMEBREW_GITHUB_API_TOKEN` and `MISE_GITHUB_TOKEN` for rate limits. `gh` gets a short-lived (1 hour) GitHub App
installation token from a fnox `github-app` lease, set as `GH_TOKEN` for one command:

```sh
fnox -P pantheon-org exec -- gh issue create --repo pantheon-org/tekhne --title "..." --body "..."
fnox -P pantheon-org exec -- gh pr list --repo pantheon-org/tekhne
fnox -P pantheon-org lease list --active   # cached token and its expiry
fnox -P pantheon-org lease revoke github   # revoke it on GitHub now
```

- The token acts as the app (issues and PRs show `pantheon-ai-bot[bot]`), not as you, and is limited to contents read,
  issues write and pull requests write, whatever else the app has. It is reused until 5 minutes before it expires.

- One token covers one installation, so each org is a fnox profile (`-P <org>`). `fnox exec` without `-P` uses
  `github_app.user`, the personal account, which is not set up yet: plain `gh` has no token.

- Config lives in `~/.config/chezmoi/init-defaults.toml` (not in the repo), then `mise run init` and `mise run apply`:

  ```toml
  [github_app.orgs.pantheon-org]
  app_id = "3181784"            # pantheon-ai-bot (private app owned by pantheon-org)
  installation_id = "118885330"
  ```

- The private key is in the macOS keychain, service `fnox`, account `GITHUB_APP_PRIVATE_KEY_<org>` (`..._KEY` for
  `github_app.user`); on Linux it is the entry's `key_age` ciphertext. CI uses a different key of the same app (org
  secret `GH_APP_PRIVATE_KEY`), so the laptop key can be revoked on its own under the app's key pairs. The app has broad
  org permissions, so treat the keychain item as high value.

- Adding another installation: generate a key pair on the app, store it without touching the global fnox config, then
  add the entry above:

  ```sh
  f=~/Downloads/<app>.private-key.pem && d=$(mktemp -d) &&
    printf '[providers.keychain]\ntype = "keychain"\nservice = "fnox"\n' > "$d/k.toml" &&
    FNOX_CONFIG_DIR="$d" fnox -c "$d/k.toml" set K --provider keychain \
      --key-name GITHUB_APP_PRIVATE_KEY_<org> --from-file "$f" && rm -rf "$d" "$f"
  ```

- The app ID and installation ID can be read from the API with a JWT signed by the key (`GET /app`,
  `GET /app/installations`), or from the app's settings page and the installation's Configure URL.
