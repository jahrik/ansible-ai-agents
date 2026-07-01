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
(`~/.gemini/config/mcp_config.json`). The **Context7** documentation server is wired up by
default; it's keyless and remote, so it works out of the box with **no token and no Docker**.
Additional servers (including GitHub) are opt-in — see [MCP Servers](#mcp-servers).

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
| `ai_agents_mcp_servers`         | list (`context7` on by default)          | MCP servers to wire into Claude Code + AGY      |

## MCP Servers

`ai_agents_mcp_servers` defines MCP servers to register with every enabled agent that
supports them. Each entry is either a remote (`type: http`) or local (`type: stdio`)
server. The default is [Context7](https://context7.com), a keyless documentation server
that fetches up-to-date, version-specific library docs into the agent's context:

```yaml
ai_agents_mcp_servers:
  - name: context7
    type: http
    url: "https://mcp.context7.com/mcp"
  # A local (stdio) server example:
  - name: my-local-server
    type: stdio
    command: npx
    args: ["-y", "@scope/some-mcp"]
    env:
      SOME_FLAG: "1"
```

**Never put a secret in this variable.** Reference credentials through an environment
variable instead — Claude Code expands `${VAR}` in urls, headers, and env values at
runtime, so only the placeholder (never the secret) is written to `~/.claude.json`.
Context7 needs no credential; for higher rate limits add a header referencing an env var:

```yaml
headers:
  CONTEXT7_API_KEY: "${CONTEXT7_API_KEY}"
```

> **AGY/Antigravity does not expand `${VAR}`.** Any server that needs a secret will not
> authenticate there — configure those by hand in AGY if you need them (never commit a
> token). Keyless servers like Context7 work in both tools.

### Adding the GitHub server (opt-in)

The GitHub MCP server is **not** a default because every variant requires a Personal
Access Token — OAuth fails Claude Code's Dynamic Client Registration requirement
(`Incompatible auth server: does not support dynamic client registration`,
[claude-code#52638](https://github.com/anthropics/claude-code/issues/52638)), and the
local Docker/binary servers only accept a PAT
([github-mcp-server#600](https://github.com/github/github-mcp-server/issues/600)). Add it
to `ai_agents_mcp_servers` yourself if you want it:

```yaml
- name: github
  type: http
  url: "https://api.githubcopilot.com/mcp/"
  headers:
    Authorization: "Bearer ${GITHUB_PERSONAL_ACCESS_TOKEN}"
```

Then supply the token via the environment:

```bash
# 1. Create a GitHub Personal Access Token (fine-grained, scoped to the repos you want
#    the agent to access): https://github.com/settings/personal-access-tokens

# 2. Export it where your shell will pick it up (e.g. ~/.bashrc, or a secrets manager).
#    Do NOT paste it into the playbook or any committed file.
export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxx

# 3. Start a fresh Claude Code session so it reads the variable, then verify.
claude mcp list
# github: https://api.githubcopilot.com/mcp/ (HTTP) - ✓ Connected
```

The token is read from the environment at launch — restart `claude` after exporting it.
If the variable is unset, the server simply reports `Failed to connect`. Re-running the
role is idempotent and does not overwrite existing servers.

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
