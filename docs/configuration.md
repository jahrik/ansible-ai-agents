# Configuration Guide

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
├── hooks/             # lifecycle guard scripts the role registers in each agent's settings
└── skills/            # modular skill packs (both tools discover these)
    └── <skill>/
        └── SKILL.md
```

## Tags

Run or skip parts of the role with tags:

```bash
ansible-playbook playbook.yml --tags ai_agents:install
ansible-playbook playbook.yml --skip-tags ai_agents:mcp_servers
```

| Tag                      | Scope                                             |
| ------------------------ | ------------------------------------------------- |
| `ai_agents`              | All role tasks                                    |
| `ai_agents:vars`         | OS-specific variable loading                      |
| `ai_agents:install`      | Agent CLI installs                                |
| `ai_agents:clone`        | Clone the agent config repo                       |
| `ai_agents:symlinks`     | Wire config repo into tool paths                  |
| `ai_agents:mcp_servers`  | Install the `mcp-servers` package                 |
| `ai_agents:mcp`          | Register MCP servers                              |
| `ai_agents:settings`     | Merge deny rules + hooks into Claude/AGY settings |
| `ai_agents:github_app`   | GitHub App env file + wrapper                     |
| `ai_agents:git_identity` | Global git identity for the App's `[bot]` account |
