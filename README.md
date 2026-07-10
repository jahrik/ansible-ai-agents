# ansible-ai-agents

[![CICD](https://github.com/jahrik/ansible-ai-agents/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-ai-agents/actions/workflows/cicd.yml)
[![Galaxy](https://img.shields.io/badge/galaxy-jahrik.ai__agents-blueviolet)](https://galaxy.ansible.com/jahrik/ai_agents)

Ansible role to install and configure AI coding agents across Linux and macOS environments.

Installs agents (AGY/Antigravity, Claude Code) and wires up a pluggable
**agent-config repo** — a separate repository of `AGENTS.md` rules, skills, and
subagents that all agents read. It clones the config to `~/.config/agents/` and
symlinks it into each tool:

- **Claude Code** — `AGENTS.md` → `~/.claude/CLAUDE.md`, `skills/` → `~/.claude/skills`, `agents/` → `~/.claude/agents`
- **AGY/Antigravity** — `AGENTS.md` → `~/.gemini/config/AGENTS.md`, `skills/` → `~/.gemini/config/skills`, `agents/` → `~/.gemini/config/agents`

It also registers shared **MCP (Model Context Protocol) servers** with the agents that
support them — Claude Code (user scope, via `claude mcp add-json`) and AGY/Antigravity
(`~/.gemini/config/mcp_config.json`). The defaults come from the
[`mcp-servers`](https://github.com/jahrik/mcp-servers) package: **`github`** (authenticates
as a **GitHub App** — writes land as `your-app[bot]`, and **no secret is written to any
config file**; the role records only the path to your PEM), **`ws`** (read-only local git
workspace surveys), **`data`** (SQL over large local files + scratch tables, DuckDB
engine), **`dispatcher`** (async agent-to-agent task delegation), **`lsp`** (language server proxy), and **`memory`** (persistent cross-session long-term memory using DuckDB). See [MCP Servers](#mcp-servers).

## Requirements

- Ansible 2.16+
- `git` on target host (to clone the agent-config repo)
- For the default `github` MCP server: a GitHub App (create one at
  _Settings → Developer settings → GitHub Apps_, install it on your repos) and its
  private-key PEM on the target host. The role deploys the identity; it can't create
  the App for you.

## Role Variables

| Variable                                    | Default                                                             | Description                                                                                                                        |
| ------------------------------------------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `ai_agents_config_repo`                     | `https://github.com/jahrik/agent-config`                            | Git URL of your agent config repo                                                                                                  |
| `ai_agents_config_ref`                      | `main`                                                              | Branch or tag to check out                                                                                                         |
| `ai_agents_config_dest`                     | `~/.config/agents`                                                  | Where the config repo is cloned                                                                                                    |
| `ai_agents_install.agy`                     | `true`                                                              | Install AGY (Antigravity CLI)                                                                                                      |
| `ai_agents_install.claude_code`             | `true`                                                              | Install Claude Code CLI                                                                                                            |
| `ai_agents_mcp_servers`                     | list (`github`/`ws`/`data`/`dispatcher`/`lsp`/`memory` by default)  | MCP servers to wire into Claude Code + AGY                                                                                         |
| `ai_agents_mcp_servers_install`             | `true`                                                              | Install the `mcp-servers` package (`uv tool`)                                                                                      |
| `ai_agents_mcp_servers_source`              | `git+https://github.com/jahrik/mcp-servers`                         | Source `uv tool install` pulls the package from                                                                                    |
| `ai_agents_mcp_servers_upgrade`             | `true`                                                              | Upgrade `mcp-servers` to the latest source ref on every run (`uv tool upgrade`, idempotent); set false to pin the installed commit |
| `ai_agents_mcp_github_app_id`               | `""`                                                                | GitHub App ID for mcp-github (set all three `_app_*` vars to enable App auth)                                                      |
| `ai_agents_mcp_github_app_installation_id`  | `""`                                                                | GitHub App installation ID                                                                                                         |
| `ai_agents_mcp_github_app_private_key_file` | `""`                                                                | Path to the App's private-key PEM (path only — the key is never copied anywhere)                                                   |
| `ai_agents_git_user_name` / `_email`        | `""`                                                                | Optional global git identity matching the App's `[bot]` account                                                                    |
| `ai_agents_mcp_workspace_root`              | `~/github`                                                          | Root the `workspace` MCP server surveys for git repos                                                                              |
| `ai_agents_claude_permission_deny`          | `["Bash(gh)", "Bash(gh:*)"]`                                        | Deny rules merged into `~/.claude/settings.json` — keeps the `gh` CLI human-only                                                   |
| `ai_agents_claude_hooks`                    | guard-bash on `Bash`; guard-write + format-on-edit on `Write\|Edit` | Hooks merged into `~/.claude/settings.json` (pre-existing hook entries preserved)                                                  |
| `ai_agents_agy_hooks`                       | guard-bash on `run_command`; guard-write on file-write tools        | Named hook groups merged into `~/.gemini/config/hooks.json`                                                                        |

## CLI Toolchain

The role optionally installs a standardized agent CLI toolchain into `~/.local/bin` (bypassing root/system packages). These are pinned static release binaries to ensure the agent has the tools it needs across all distributions (and works on immutable rootfs).

See [docs/cli-toolchain.md](docs/cli-toolchain.md) for full details on included tools and upgrade instructions.

## MCP Servers

`ai_agents_mcp_servers` defines MCP servers to register with every enabled agent that supports them. Each entry is either a local (`type: stdio`) or remote (`type: http`) server.

By default, it installs the `mcp-servers` package (`github`, `ws`, `data`, `dispatcher`, `lsp`, `memory`).

See [docs/mcp-servers.md](docs/mcp-servers.md) for full documentation on configuration, adding new servers, and GitHub App authentication.

## Configuration

See [docs/configuration.md](docs/configuration.md) for details on:

- Using tags to run specific parts of the role
- Pointing the role to your own agent config repository

## Example Playbook

```yaml
- hosts: workstations
  roles:
    - role: jahrik.ai_agents
      vars:
        ai_agents_config_repo: "https://github.com/jahrik/agent-config"
        ai_agents_install:
          agy: true
          claude_code: true
```

## Composing with another role

```yaml
# requirements.yml
- src: jahrik.ai_agents
```

## Testing

```bash
uv sync
source .venv/bin/activate
yamllint .
ansible-lint
molecule test
```

## License

MIT

## Author

[jahrik](https://github.com/jahrik)
