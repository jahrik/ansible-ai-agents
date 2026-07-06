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
(`~/.gemini/config/mcp_config.json`). The defaults come from the
[`mcp-servers`](https://github.com/jahrik/mcp-servers) package: **`github`** (authenticates
as a **GitHub App** — writes land as `your-app[bot]`, and **no secret is written to any
config file**; the role records only the path to your PEM), **`ws`** (read-only local git
workspace surveys), and **`data`** (SQL over large local files + scratch tables, DuckDB
engine). See [MCP Servers](#mcp-servers).

## Requirements

- Ansible 2.16+
- `git` on target host (to clone the agent-config repo)
- For the default `github` MCP server: a GitHub App (create one at
  _Settings → Developer settings → GitHub Apps_, install it on your repos) and its
  private-key PEM on the target host. The role deploys the identity; it can't create
  the App for you.

## Role Variables

| Variable                                    | Default                                            | Description                                                                              |
| ------------------------------------------- | -------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `ai_agents_config_repo`                     | `https://github.com/jahrik/agent-config`           | Git URL of your agent config repo                                                        |
| `ai_agents_config_ref`                      | `main`                                             | Branch or tag to check out                                                               |
| `ai_agents_config_dest`                     | `~/.config/agents`                                 | Where the config repo is cloned                                                          |
| `ai_agents_install.agy`                     | `true`                                             | Install AGY (Antigravity CLI)                                                            |
| `ai_agents_install.claude_code`             | `true`                                             | Install Claude Code CLI                                                                  |
| `ai_agents_install.aider`                   | `false`                                            | Install Aider CLI                                                                        |
| `ai_agents_install.codex`                   | `false`                                            | Install Codex CLI                                                                        |
| `ai_agents_install.cursor`                  | `false`                                            | Install Cursor — _planned, not yet implemented_                                          |
| `ai_agents_mcp_servers`                     | list (`github`/`ws`/`data` by default)             | MCP servers to wire into Claude Code + AGY                                               |
| `ai_agents_mcp_servers_install`             | `true`                                             | Install the `mcp-servers` package (`uv tool`)                                            |
| `ai_agents_mcp_servers_source`              | `git+https://github.com/jahrik/mcp-servers@v0.3.0` | Source `uv tool install` pulls the package from                                          |
| `ai_agents_mcp_servers_upgrade`             | `false`                                            | Force-reinstall `mcp-servers` to pull the latest commit (install is otherwise once-only) |
| `ai_agents_mcp_github_app_id`               | `""`                                               | GitHub App ID for mcp-github (set all three `_app_*` vars to enable App auth)            |
| `ai_agents_mcp_github_app_installation_id`  | `""`                                               | GitHub App installation ID                                                               |
| `ai_agents_mcp_github_app_private_key_file` | `""`                                               | Path to the App's private-key PEM (path only — the key is never copied anywhere)         |
| `ai_agents_git_user_name` / `_email`        | `""`                                               | Optional global git identity matching the App's `[bot]` account                          |
| `ai_agents_mcp_workspace_root`              | `~/github`                                         | Root the `workspace` MCP server surveys for git repos                                    |
| `ai_agents_claude_permission_deny`          | `["Bash(gh)", "Bash(gh:*)"]`                       | Deny rules merged into `~/.claude/settings.json` — keeps the `gh` CLI human-only         |
| `ai_agents_claude_hooks`                    | guard-bash on `Bash`                               | Hooks merged into `~/.claude/settings.json` (pre-existing hook entries preserved)        |
| `ai_agents_agy_hooks`                       | guard-bash on `run_command`                        | Named hook groups merged into `~/.gemini/config/hooks.json`                              |

## CLI Toolchain

The role optionally installs a standardized agent CLI toolchain into `~/.local/bin` (bypassing root/system packages). These are pinned static release binaries to ensure the agent has the tools it needs across all distributions (and works on immutable rootfs).

- **Tools:** `rg`, `fd`, `jq`, `yq`, `bat`, `delta`, `ast-grep`, `xsv`, `htmlq`, `gron`, `jc`, `sd`, `tokei`, `duckdb`
- **Linters:** `shellcheck`, `shfmt`, `hadolint`, `actionlint`
- **Utilities:** `repo-sync`

This is controlled by `ai_agents_install.cli_tools` (and `ai_agents_install.repo_sync`) in your variables.

### Upgrading Tools

The role is idempotent — it skips downloads if the binary already exists. To upgrade a tool, bump its pinned version in `defaults/main.yml` and delete the old binary from `~/.local/bin` before running the playbook.

## MCP Servers

`ai_agents_mcp_servers` defines MCP servers to register with every enabled agent that
supports them. Each entry is either a local (`type: stdio`) or remote (`type: http`)
server. The default is **`github`**, from the
[`mcp-servers`](https://github.com/jahrik/mcp-servers) package — a local stdio server that
authenticates as a **GitHub App**. Point the role at your App's identity:

```yaml
ai_agents_mcp_github_app_id: "123456"
ai_agents_mcp_github_app_installation_id: "654321"
ai_agents_mcp_github_app_private_key_file: "{{ ansible_env.HOME }}/.ssh/my-app.private-key.pem"
```

The App ID and installation ID are not secrets, and the private key **stays in the PEM
where it already lives** — the role writes only its _path_ into
`~/.config/ai-agents/github-app.env` (mode `0600`) and registers the server through a
`~/.local/bin/mcp-github-app` wrapper that exports the key at launch. No agent config
file ever holds the key. Optionally set `ai_agents_git_user_name` /
`ai_agents_git_user_email` so local commits match the App's `[bot]` identity.

The role installs the package with `uv tool install` (bootstrapping `uv` if needed); set
`ai_agents_mcp_servers_install: false` to skip that. Leave the `ai_agents_mcp_github_app_*`
vars unset and the bare server is registered anyway — it just can't authenticate until the
`GITHUB_APP_*` env vars reach it some other way.

The same package provides the read-only **`ws`** server (`mcp-workspace`), registered by
default: local git surveys (dirty trees, unpushed work, stale branches) across
`ai_agents_mcp_workspace_root` (default `~/github`). No credentials needed. It registers as
`ws` because Claude Code reserves the name `workspace`.

### Adding other servers

Add remote (`type: http`) or local (`type: stdio`) entries to the list:

```yaml
ai_agents_mcp_servers:
  - name: context7 # keyless docs server
    type: http
    url: "https://mcp.context7.com/mcp"
  - name: my-local-server
    type: stdio
    command: npx
    args: ["-y", "@scope/some-mcp"]
```

**Never put a secret in this variable.** For servers that need a credential, reference an
environment variable — Claude Code expands `${VAR}` in urls, headers, and env values at
runtime, so only the placeholder (never the secret) is written to `~/.claude.json`.

> **AGY/Antigravity does not expand `${VAR}`.** A secret-bearing server won't authenticate
> there — configure those by hand in AGY if you need them (never commit a token).

## Tags

Run or skip parts of the role with tags:

```bash
ansible-playbook playbook.yml --tags ai_agents:install
ansible-playbook playbook.yml --skip-tags ai_agents:mcp_servers
```

| Tag                     | Scope                             |
| ----------------------- | --------------------------------- |
| `ai_agents`             | All role tasks                    |
| `ai_agents:vars`        | OS-specific variable loading      |
| `ai_agents:install`     | Agent CLI installs                |
| `ai_agents:clone`       | Clone the agent config repo       |
| `ai_agents:symlinks`    | Wire config repo into tool paths  |
| `ai_agents:mcp_servers` | Install the `mcp-servers` package |
| `ai_agents:mcp`         | Register MCP servers              |

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
