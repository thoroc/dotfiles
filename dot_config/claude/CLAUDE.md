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

## Style Preferences
- No emojis unless explicitly requested.
- Short, direct responses. Lead with the answer.
- Reference file paths with `file:line_number` format.
