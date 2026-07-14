# ansible-ai-agents

[![CICD](https://github.com/jahrik/ansible-ai-agents/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-ai-agents/actions/workflows/cicd.yml)
[![Galaxy](https://img.shields.io/badge/galaxy-jahrik.ai__agents-blueviolet)](https://galaxy.ansible.com/jahrik/ai_agents)

This Ansible role is the deployment mechanism for the **Agent Harness** - a standardized, fully-equipped AI coding agent environment on Linux and macOS.

It acts as the glue that ties together the three parts of the harness:

1. **This Role (`jahrik/ansible-ai-agents`)** - Installs the agent CLIs (AGY and Claude Code) and a standardized CLI toolchain.
2. **[jahrik/agent-config](https://github.com/jahrik/agent-config)** - The centralized repository of rules (`AGENTS.md`), personas, and skills.
3. **[jahrik/mcp-servers](https://github.com/jahrik/mcp-servers)** - The Model Context Protocol servers that give agents access to GitHub, workspaces, databases, and cross-session memory.

## Features

- **Installs Agents:** Sets up AGY (Antigravity) and Claude Code.
- **Shared Configuration:** Clones the central `agent-config` repository (containing `AGENTS.md`, `skills/`, and `agents/`) and symlinks it into both agents.
- **MCP Servers:** Wires up a suite of Model Context Protocol servers:
  - `github` - App-authenticated GitHub access (no raw tokens).
  - `ws` - Local git workspace surveys.
  - `data` - SQL over local files (DuckDB).
  - `dispatcher` - Async task delegation between agents.
  - `lsp` - Language-server code navigation.
  - `memory` - Persistent cross-session memory (DuckDB).
  - `playwright` - Local browser rendering via Chromium (Arch/SteamOS needs manual shared library deps).
- **CLI Toolchain:** Deploys pinned, statically-compiled CLI tools (`rg`, `fd`, `jq`, linters, etc.) to guarantee a predictable environment for the agents.

## Documentation

- [Role Variables](docs/variables.md) - Full list of configuration variables and their defaults.
- [CLI Toolchain](docs/cli-toolchain.md) - Included tools and upgrade instructions.
- [MCP Servers](docs/mcp-servers.md) - Configuration, adding custom servers, and GitHub App auth.
- [Configuration & Tags](docs/configuration.md) - Using your own config repo and running specific Ansible tags.

## Requirements

- **Ansible:** 2.16+
- **Git:** Required on the target host.
- **GitHub App _(Optional)_:** A GitHub App and its private key PEM file on the target host, required only if using the default `github` MCP server.

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
