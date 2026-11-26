# Global Claude Instructions

## Package Management
- Default to `bun` for JS/TS projects. Use `pnpm` only if the project explicitly requires it.
- Never use bare `npm install` — it is aliased to `npq-hero` (security proxy) and will behave unexpectedly. Use `bun add` or `pnpm add` instead.

## File Search
- `fd` is available for file discovery (installed 2026-03-06 via mise).
- `rg` (ripgrep) is available for content search.

## Shell / Environment
- Managed via `mise`. Most tools are in `~/.local/share/mise/installs/`.
- `uv`/`uvx` is available for Python tooling.
- `lefthook` is used for git hooks.

## GitHub (`gh`)
- No GitHub token is exported to the shell: plain `gh` is not logged in. Do not ask for, create or widen a PAT.
- For pantheon-org repos, run gh through the fnox GitHub App lease: `fnox -P pantheon-org exec -- gh <args>`. It mints a 1-hour `pantheon-ai-bot` installation token as `GH_TOKEN`.
- The token can read contents and write issues and pull requests. It cannot push, merge or change settings; push with git over SSH as the user instead.
- Issues, comments and PRs created this way show as `pantheon-ai-bot[bot]`, not as the user. Say so before creating them.
- Other orgs and the personal account have no lease yet; if gh needs one, say so instead of working around it. Setup: "GitHub tokens" in `~/.dotfiles/README.md`.

## Style Preferences
- No emojis unless explicitly requested.
- Short, direct responses. Lead with the answer.
- Reference file paths with `file:line_number` format.

## Blast Radius
Before changing, adding, removing or gating anything, work out its blast radius and say what you checked. Do not stop at
the line you were asked to touch:
- Find every consumer: search the whole repo for the key, tool, file or name being changed (code, templates, scripts,
  configs, ignore files, docs). Whatever exists only to support it must change with it.
- Check what it assumes about the environment: OS, architecture, installed tools, and the PATH of whatever runs it.
  Shells, GUI clients, git hooks and install scripts do not all load your shell setup. Check for duplicate or stale
  binaries (`type -a <tool>`).
- Check who else is affected: existing installs and stored settings, defaults that must keep the old behaviour, and
  generated files that another tool overwrites (fix the source, not the output).
- Verify every branch: exercise the changed path and its alternatives (flag on and off, each OS or architecture, tool
  present and missing) in the same minimal environment where failures occur.
- Keep changes scoped: one logical change per commit, and commit only the files that belong to it (`git commit -o <files>`).
