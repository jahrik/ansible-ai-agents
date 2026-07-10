# MCP Servers

`ai_agents_mcp_servers` defines MCP servers to register with every enabled agent that
supports them. Each entry is either a local (`type: stdio`) or remote (`type: http`)
server. The default is **`github`**, from the
[`mcp-servers`](https://github.com/jahrik/mcp-servers) package - a local stdio server that
authenticates as a **GitHub App**. Point the role at your App's identity:

```yaml
ai_agents_mcp_github_app_id: "123456"
ai_agents_mcp_github_app_installation_id: "654321"
ai_agents_mcp_github_app_private_key_file: "{{ ansible_env.HOME }}/.ssh/my-app.private-key.pem"
```

The App ID and installation ID are not secrets, and the private key **stays in the PEM
where it already lives** - the role writes only its _path_ into
`~/.config/ai-agents/github-app.env` (mode `0600`) and registers the server through a
`~/.local/bin/mcp-github-app` wrapper that exports the key at launch. No agent config
file ever holds the key. Optionally set `ai_agents_git_user_name` /
`ai_agents_git_user_email` so local commits match the App's `[bot]` identity.

The role installs the package with `uv tool install` (bootstrapping `uv` if needed); set
`ai_agents_mcp_servers_install: false` to skip that. Leave the `ai_agents_mcp_github_app_*`
vars unset and the bare server is registered anyway - it just can't authenticate until the
`GITHUB_APP_*` env vars reach it some other way.

The same package provides the other five default servers, none of which need credentials:

- **`ws`** (`mcp-workspace`) - read-only local git surveys (dirty trees, unpushed work, stale
  branches) across `ai_agents_mcp_workspace_root` (default `~/github`). It registers as `ws`
  because Claude Code reserves the name `workspace`.
- **`data`** (`mcp-data`) - SQL over local files (CSV/JSON/JSONL/Parquet) via DuckDB.
- **`dispatcher`** (`mcp-dispatcher`) - async agent-to-agent job queue (SQLite-backed).
- **`lsp`** (`mcp-lsp`) - language-server and tree-sitter code navigation.
- **`memory`** (`mcp-memory`) - persistent, cross-session long-term memory store using DuckDB.

Each server has a page in the package's
[`docs/`](https://github.com/jahrik/mcp-servers/tree/main/docs).

## Adding other servers

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
environment variable - Claude Code expands `${VAR}` in urls, headers, and env values at
runtime, so only the placeholder (never the secret) is written to `~/.claude.json`.

> **AGY/Antigravity does not expand `${VAR}`.** A secret-bearing server won't authenticate
> there - configure those by hand in AGY if you need them (never commit a token).
