# AGENTS.md

This is an Ansible role. AI agents working in this repo should follow these conventions.

## Purpose

Installs and configures AI coding agents across OS environments. See `README.md` for usage.

## Key Files

- `defaults/main.yml` — all tuneable variables with defaults
- `tasks/main.yml` — entry point; delegates to `install/` and `config/` sub-tasks
- `tasks/install/<agent>.yml` — one file per agent
- `tasks/config/clone.yml` — clones the agent config repo
- `tasks/config/symlinks.yml` — wires config repo into tool-expected locations
- `molecule/default/` — test scenario (converge + verify)

## Conventions

- Use FQCN for all Ansible modules (`ansible.builtin.*`)
- New agents: add a task file at `tasks/install/<agent_name>.yml` and a key in `defaults/main.yml`
- Test with `mtest converge && mtest verify`
- Never hardcode credentials or paths — use variables from `defaults/main.yml`
- CI runs on every PR via GitHub Actions (yamllint → molecule → galaxy publish on merge)
