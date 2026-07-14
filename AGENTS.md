# AGENTS.md

This is an Ansible role. AI agents working in this repo should follow these conventions.

## Purpose

Installs and configures AI coding agents across OS environments. See `README.md` for usage.

## Key Variables

| Variable                | Default                                                                            | Description                                  |
| ----------------------- | ---------------------------------------------------------------------------------- | -------------------------------------------- |
| `ai_agents_config_repo` | `https://github.com/jahrik/agent-config`                                           | Git URL of the agent config repo to clone    |
| `ai_agents_config_ref`  | `main`                                                                             | Branch or tag to check out                   |
| `ai_agents_config_dest` | `~/.config/agents`                                                                 | Clone destination for the config repo        |
| `ai_agents_install`     | dict (`agy`/`claude_code` on by default)                                           | Per-agent install toggles                    |
| `ai_agents_symlinks`    | list                                                                               | `src`→`dest` links from config repo to tools |
| `ai_agents_mcp_servers` | list (7 by default: `github`/`ws`/`data`/`dispatcher`/`lsp`/`memory`/`playwright`) | MCP servers wired into Claude Code + AGY     |

## Key Files

- `defaults/main.yml` — all tuneable variables with defaults
- `tasks/main.yml` — entry point; delegates to `install/` and `config/` sub-tasks
- `tasks/install/<agent>.yml` — one file per agent
- `tasks/config/clone.yml` — clones the agent config repo
- `tasks/config/symlinks.yml` — wires config repo into tool-expected locations
- `tasks/config/mcp_servers.yml` — installs the `mcp-servers` package (via `uv tool`)
- `tasks/config/mcp.yml` — registers MCP servers (Claude Code via CLI, AGY via JSON)
- `tasks/config/claude_settings.yml` — merges deny rules + hooks into `~/.claude/settings.json`
- `tasks/config/agy_hooks.yml` — merges managed hook groups into `~/.gemini/config/hooks.json`
- `tasks/config/github_app.yml` — GitHub App env file (`0600`) + launch wrapper
- `tasks/config/git_identity.yml` — optional global git identity for the App's `[bot]` account
- `templates/mcp_config.json.j2` — renders/merges AGY's `mcp_config.json`
- `molecule/default/` — test scenario (converge + verify)

## Conventions

- Use FQCN for all Ansible modules (`ansible.builtin.*`)
- New agents: add a task file at `tasks/install/<agent_name>.yml` and a key in `defaults/main.yml`
- Test with:
  ```bash
  uv sync
  source .venv/bin/activate
  yamllint .
  ansible-lint
  molecule test
  ```
- Never hardcode credentials or paths — use variables from `defaults/main.yml`
- CI runs on every PR via GitHub Actions (yamllint → molecule → galaxy publish on merge)
