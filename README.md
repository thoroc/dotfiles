# Chezmoi dotfiles

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Prerequisite](#prerequisite)
- [Setup](#setup)
- [What gets applied](#what-gets-applied)
- [Claude Code config](#claude-code-config)
- [Day-to-day](#day-to-day)
- [GitHub tokens](#github-tokens)
  - [How it works](#how-it-works)
  - [Set up an account](#set-up-an-account)

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

Git hooks are managed by [hk](https://hk.jdx.dev/) (`hk.pkl`): [betterleaks](https://github.com/betterleaks/betterleaks)
scans every staged change for secrets, and prettier, rumdl and doctoc run on markdown files. CI
(`.github/workflows/secret-scan.yml`) runs two independent scanners, both pinned in `mise.toml`: betterleaks on the
commits each push or PR adds (the full history weekly), and [kingfisher](https://github.com/mongodb/kingfisher) on the
full history every run, which also checks whether a secret it finds is still live.

## GitHub tokens

No long-lived GitHub token is exported to the shell, so plain `gh` is not logged in. A classic PAT (no scopes needed)
can be stored for rate limits; it is exported only as `HOMEBREW_GITHUB_API_TOKEN` and `MISE_GITHUB_TOKEN`, never as
`GITHUB_TOKEN`. `gh` instead gets a short-lived (1 hour) GitHub App installation token from a
[fnox `github-app` lease](https://github.com/jdx/fnox/blob/main/docs/leases/github-app.md), set as `GH_TOKEN` for one
command only.

### How it works

- Each account you want tokens for (your personal account, or an org) is one entry in
  `~/.config/chezmoi/init-defaults.toml`, pointing at a GitHub App installed on that account. The entry for your
  personal account is `github_app.user` and is used by plain `fnox exec`; each org is `github_app.orgs.<org>` and
  becomes a fnox profile, used with `fnox -P <org> exec`. One token only ever covers one installation.
- The token acts as the app, not as you: issues, comments and PRs show the author `<app-slug>[bot]`. It is narrowed to
  contents read, issues write and pull requests write, whatever else the app is allowed, so it cannot push, merge or
  change settings. Push with git over SSH as yourself.
- The token is cached and reused until 5 minutes before it expires. An account with no entry gets no lease: the command
  still runs, unauthenticated.

```sh
fnox -P <org> exec -- gh pr list --repo <org>/<repo>
fnox -P <org> exec -- gh issue create --repo <org>/<repo> --title "..." --body "..."
fnox exec -- gh repo list               # personal account (github_app.user)
fnox -P <org> lease list --active       # cached token and its expiry
fnox -P <org> lease revoke github       # revoke the cached token on GitHub now
```

### Set up an account

Do this once per account (`<owner>` is your username or the org). Creating, installing and keying an app has no API, so
steps 1 to 3 are in the browser; the URL below fills in the form for you.

1. Create a private GitHub App owned by that account. Open the URL (for your personal account, use
   `https://github.com/settings/apps/new?...` with the same parameters), change the name if it is taken (names are
   global), and click **Create GitHub App**:

   ```text
   https://github.com/organizations/<owner>/settings/apps/new?name=<owner>-cli-bot&url=https://github.com/<owner>&public=false&webhook_active=false&contents=read&issues=write&pull_requests=write
   ```

   Keep "Only on this account" (the default for `public=false`): an app that any account can install gains nothing here.
   You can reuse an app the account already has instead, if it has at least those three permissions; the lease asks for
   no more than that. Note the app's slug (the last part of its settings URL): it is the bot's login.

2. Install it: on the app's page, **Install App**, choose the account, and pick all repositories or just the ones you
   want. The installation ID is the number at the end of the URL you land on
   (`.../settings/installations/<installation_id>`). The app ID is on the app's **General** page.

3. Create a key pair: on the app's **General** page, under **Private keys** (shown as "Credentials as Key Pairs" on some
   accounts), **Generate a private key**. A `.pem` file downloads. Give each machine its own key pair, separate from any
   CI key, so you can revoke one without the others.

4. Store the key and delete the download. On macOS it goes into the keychain (service `fnox`, account
   `GITHUB_APP_PRIVATE_KEY_<org>`, or `GITHUB_APP_PRIVATE_KEY` for your personal account) through a throwaway fnox
   config, so the global one is not touched and the key is never printed:

   ```sh
   f=~/Downloads/<app-slug>.<date>.private-key.pem && d=$(mktemp -d) &&
     printf '[providers.keychain]\ntype = "keychain"\nservice = "fnox"\n' > "$d/k.toml" &&
     FNOX_CONFIG_DIR="$d" fnox -c "$d/k.toml" set K --provider keychain \
       --key-name GITHUB_APP_PRIVATE_KEY_<org> --from-file "$f" && rm -rf "$d" "$f"
   ```

   On Linux it is stored as an age ciphertext for your `secrets.age_recipient` instead. This prints the value for
   `key_age` in step 5 (ciphertext only, safe to keep in `init-defaults.toml`):

   ```sh
   f=~/Downloads/<app-slug>.<date>.private-key.pem && d=$(mktemp -d) &&
     printf '[providers.age]\ntype = "age"\nrecipients = ["%s"]\n' "<age_recipient>" > "$d/k.toml" &&
     FNOX_CONFIG_DIR="$d" fnox -c "$d/k.toml" set K --provider age --from-file "$f" &&
     sed -n 's/.*value = "\([^"]*\)".*/\1/p' "$d/k.toml" && rm -rf "$d" "$f"
   ```

5. Add the entry to `~/.config/chezmoi/init-defaults.toml` (it is outside the repo; the IDs are not secret, but they
   stay out of a public repo). See `init-defaults.example.toml`:

   ```toml
   [github_app.orgs.<org>]          # or [github_app.user] for your personal account
   app_id = "<app_id>"
   installation_id = "<installation_id>"
   slug = "<app-slug>"              # optional: names the bot in ~/.claude/CLAUDE.md
   # key_age = "<ciphertext>"       # Linux only, from step 4
   ```

6. Render it and check the token works:

   ```sh
   mise run init && mise run apply
   fnox -P <org> exec -- gh auth status    # shows <app-slug>[bot] (GH_TOKEN)
   ```

To stop using an account, delete its entry and run step 6, then delete the keychain item and the key pair on the app.
The keychain item and the key can mint tokens with every permission the app has, so treat them as high value.
