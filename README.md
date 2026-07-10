# ansible-ai-agents

[![CICD](https://github.com/jahrik/ansible-ai-agents/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-ai-agents/actions/workflows/cicd.yml)
[![Galaxy](https://img.shields.io/badge/galaxy-jahrik.ai__agents-blueviolet)](https://galaxy.ansible.com/jahrik/ai_agents)

An Ansible role to deploy a standardized, fully-equipped AI coding agent environment on Linux and macOS.

## Features

- **Installs Agents:** Sets up AGY (Antigravity) and Claude Code.
- **Shared Configuration:** Clones a central `agent-config` repository (containing `AGENTS.md`, `skills/`, and `agents/`) and symlinks it into both agents.
- **MCP Servers:** Wires up a suite of Model Context Protocol servers:
  - `github` — App-authenticated GitHub access (no raw tokens).
  - `ws` — Local git workspace surveys.
  - `data` — SQL over local files (DuckDB).
  - `dispatcher` — Async task delegation between agents.
  - `lsp` — Language-server code navigation.
  - `memory` — Persistent cross-session memory (DuckDB).
- **CLI Toolchain:** Deploys pinned, statically-compiled CLI tools (`rg`, `fd`, `jq`, linters, etc.) to guarantee a predictable environment for the agents.

## Documentation

- [CLI Toolchain](docs/cli-toolchain.md) — Included tools and upgrade instructions.
- [MCP Servers](docs/mcp-servers.md) — Configuration, adding custom servers, and GitHub App auth.
- [Configuration & Tags](docs/configuration.md) — Using your own config repo and running specific Ansible tags.

## Requirements

- **Ansible:** 2.16+
- **Git:** Required on the target host.
- **GitHub App _(Optional)_:** A GitHub App and its private key PEM file on the target host, required only if using the default `github` MCP server.

## Role Variables

| Variable                                    | Default                                  | Description                                        |
| ------------------------------------------- | ---------------------------------------- | -------------------------------------------------- |
| `ai_agents_config_repo`                     | `https://github.com/jahrik/agent-config` | Git URL of your agent config repo.                 |
| `ai_agents_config_ref`                      | `main`                                   | Branch or tag to check out.                        |
| `ai_agents_config_dest`                     | `~/.config/agents`                       | Path where the config repo is cloned.              |
| `ai_agents_install.agy`                     | `true`                                   | Install AGY (Antigravity CLI).                     |
| `ai_agents_install.claude_code`             | `true`                                   | Install Claude Code CLI.                           |
| `ai_agents_mcp_servers`                     | _(list of 6 defaults)_                   | MCP servers to wire into Claude Code & AGY.        |
| `ai_agents_mcp_servers_install`             | `true`                                   | Install the `mcp-servers` package via `uv tool`.   |
| `ai_agents_mcp_servers_source`              | `git+https://...`                        | Source URL for the `mcp-servers` package.          |
| `ai_agents_mcp_servers_upgrade`             | `true`                                   | Run `uv tool upgrade` on every run.                |
| `ai_agents_mcp_github_app_id`               | `""`                                     | GitHub App ID for `mcp-github`.                    |
| `ai_agents_mcp_github_app_installation_id`  | `""`                                     | GitHub App Installation ID.                        |
| `ai_agents_mcp_github_app_private_key_file` | `""`                                     | Path to the App's private-key PEM.                 |
| `ai_agents_git_user_name` / `_email`        | `""`                                     | Global git identity for the App's `[bot]` account. |
| `ai_agents_mcp_workspace_root`              | `~/github`                               | Root directory for the `ws` MCP server surveys.    |
| `ai_agents_claude_permission_deny`          | `["Bash(gh)", "Bash(gh:*)"]`             | Deny rules injected into Claude settings.          |
| `ai_agents_claude_hooks`                    | _(Bash / Write guards)_                  | Guard scripts injected into Claude settings.       |
| `ai_agents_agy_hooks`                       | _(Bash / Write guards)_                  | Guard scripts injected into AGY hooks.             |

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
