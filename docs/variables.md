# Role Variables

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
