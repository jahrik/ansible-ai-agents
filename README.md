# ansible-ai-agents

[![Molecule](https://github.com/jahrik/ansible-ai-agents/actions/workflows/molecule.yml/badge.svg)](https://github.com/jahrik/ansible-ai-agents/actions/workflows/molecule.yml)
[![Galaxy](https://img.shields.io/badge/galaxy-jahrik.ai__agents-blueviolet)](https://galaxy.ansible.com/jahrik/ai_agents)

Ansible role to install and configure AI coding agents across Linux and macOS environments.

Installs agents (AGY/Antigravity, Claude Code, GitHub Copilot, Cursor) and wires up a
pluggable **agent-config repo** — a separate repository of `AGENTS.md` rules and skills
that all agents read.

## Requirements

- Ansible 2.14+
- `git` on target host
- `npm` on target host (for agent CLI installs)
- `gh` CLI on target host (for Copilot extension)

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `ai_agents_config_repo` | `https://github.com/jahrik/agent-config` | Git URL of your agent config repo |
| `ai_agents_config_ref` | `main` | Branch or tag to check out |
| `ai_agents_config_dest` | `~/.config/agents` | Where the config repo is cloned |
| `ai_agents_install.agy` | `true` | Install AGY (Antigravity CLI) |
| `ai_agents_install.claude_code` | `true` | Install Claude Code CLI |
| `ai_agents_install.copilot` | `false` | Install GitHub Copilot CLI extension |
| `ai_agents_install.cursor` | `false` | Install Cursor (AppImage/flatpak) |

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
├── skills/            # modular skill packs (AGY auto-discovers these)
│   └── <skill>/
│       └── SKILL.md
└── rules/             # additional rule files
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
          copilot: false
          cursor: false
```

## Composing with ansible-arch-workstation

```yaml
# requirements.yml
- src: jahrik.ai_agents
```

## Testing

Uses [Molecule](https://molecule.readthedocs.io/) with Docker driver via Podman:

```bash
mtest converge
mtest verify
mtest destroy
```

## License

MIT

## Author

[jahrik](https://github.com/jahrik)
