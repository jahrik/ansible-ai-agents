# ansible-ai-agents

[![CICD](https://github.com/jahrik/ansible-ai-agents/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-ai-agents/actions/workflows/cicd.yml)
[![Galaxy](https://img.shields.io/badge/galaxy-jahrik.ai__agents-blueviolet)](https://galaxy.ansible.com/jahrik/ai_agents)

Ansible role to install and configure AI coding agents across Linux and macOS environments.

Installs agents (AGY/Antigravity, Claude Code, Aider, Codex CLI) and wires up a pluggable
**agent-config repo** — a separate repository of `AGENTS.md` rules, skills, and
subagents that all agents read. It clones the config to `~/.config/agents/` and
symlinks it into each tool:

- **Claude Code** — `AGENTS.md` → `~/.claude/CLAUDE.md`, `skills/` → `~/.claude/skills`, `agents/` → `~/.claude/agents`
- **AGY/Antigravity** — `AGENTS.md` → `~/.gemini/config/AGENTS.md`, `skills/` → `~/.gemini/config/skills`, `agents/` → `~/.gemini/config/agents`
- **Aider** — generates `~/.aider.conf.yml` to read `~/.config/agents/AGENTS.md`

It also registers shared **MCP (Model Context Protocol) servers** with the agents that
support them — Claude Code (user scope, via `claude mcp add-json`) and AGY/Antigravity
(`~/.gemini/config/mcp_config.json`). The GitHub server is wired up by default and
authenticates over OAuth, so **no token is ever written to a file**.

## Requirements

- Ansible 2.16+
- `git` on target host (to clone the agent-config repo)

## Role Variables

| Variable                        | Default                                  | Description                                     |
| ------------------------------- | ---------------------------------------- | ----------------------------------------------- |
| `ai_agents_config_repo`         | `https://github.com/jahrik/agent-config` | Git URL of your agent config repo               |
| `ai_agents_config_ref`          | `main`                                   | Branch or tag to check out                      |
| `ai_agents_config_dest`         | `~/.config/agents`                       | Where the config repo is cloned                 |
| `ai_agents_install.agy`         | `true`                                   | Install AGY (Antigravity CLI)                   |
| `ai_agents_install.claude_code` | `true`                                   | Install Claude Code CLI                         |
| `ai_agents_install.aider`       | `false`                                  | Install Aider CLI                               |
| `ai_agents_install.codex`       | `false`                                  | Install Codex CLI                               |
| `ai_agents_install.cursor`      | `false`                                  | Install Cursor — _planned, not yet implemented_ |
| `ai_agents_mcp_servers`         | list (`github` on by default)            | MCP servers to wire into Claude Code + AGY      |

## MCP Servers

`ai_agents_mcp_servers` defines MCP servers to register with every enabled agent that
supports them. Each entry is either a remote (`type: http`) or local (`type: stdio`)
server:

```yaml
ai_agents_mcp_servers:
  - name: github
    type: http
    url: "https://api.githubcopilot.com/mcp/"
  - name: my-local-server
    type: stdio
    command: npx
    args: ["-y", "@scope/some-mcp"]
    env:
      SOME_FLAG: "1"
```

**Never put a secret in this variable.** The default GitHub server uses OAuth (browser
prompt on first use), so no credential is stored. To use a Personal Access Token with
Claude Code instead, reference an env var in a header — Claude Code expands `${VAR}` at
runtime (AGY does not, so leave it on OAuth):

```yaml
- name: github
  type: http
  url: "https://api.githubcopilot.com/mcp/"
  headers:
    Authorization: "Bearer ${GITHUB_PERSONAL_ACCESS_TOKEN}"
```

## Bring Your Own Config Repo

Point the role at your own fork of `agent-config`:

```yaml
# group_vars/all.yml or playbook vars
ai_agents_config_repo: "https://github.com/YOUR_USERNAME/agent-config"
```

The config repo should follow this structure:

```
agent-config/
├── AGENTS.md          # global rules — all agents read this
├── agents/            # subagent personas → ~/.claude/agents/ + ~/.gemini/config/agents/
│   └── <agent>.md
└── skills/            # modular skill packs (both tools discover these)
    └── <skill>/
        └── SKILL.md
```

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
          aider: false
          codex: false
          cursor: false
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
