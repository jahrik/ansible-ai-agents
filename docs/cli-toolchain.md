# CLI Toolchain

The role optionally installs a standardized agent CLI toolchain into `~/.local/bin` (bypassing root/system packages). These are pinned static release binaries to ensure the agent has the tools it needs across all distributions (and works on immutable rootfs).

- **Tools:** `rg`, `fd`, `jq`, `yq`, `bat`, `delta`, `gron`, `sd`, `tokei`
- **Linters:** `shellcheck`, `shfmt`, `hadolint`, `actionlint`
- **Utilities:** `rs`

This is controlled by `ai_agents_install.cli_tools` (and `ai_agents_install.repo_sync`) in your variables.

## Upgrading Tools

The role is idempotent — it skips downloads if the binary already exists. To upgrade a tool, bump its pinned version in `defaults/main.yml` and delete the old binary from `~/.local/bin` before running the playbook.
