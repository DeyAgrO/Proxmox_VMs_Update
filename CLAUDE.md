# CLAUDE.md — Proxmox_VMs_Update

This file provides context for AI assistants (Claude Code and others) working on this repository.

## Project Overview

**Proxmox_VMs_Update** is an Ansible role designed to automate the update process for Proxmox VE (Virtual Environment) hosts and their managed virtual machines (VMs) and LXC containers. The role handles package updates, kernel upgrades, and optional reboots for Debian-based Proxmox systems.

- **Language/Tech:** YAML (Ansible role), Jinja2 templating
- **Target platform:** Proxmox VE (Debian-based Linux)
- **License:** Apache 2.0

---

## Repository Structure

This project follows the standard Ansible Galaxy role layout:

```
Proxmox_VMs_Update/
├── CLAUDE.md             # This file
├── README.md             # Human-facing documentation
├── LICENSE               # Apache 2.0
├── defaults/
│   └── main.yml          # Default variable values (overridable by users)
├── vars/
│   └── main.yml          # Internal role variables (not intended for override)
├── tasks/
│   └── main.yml          # Main task entry point; may include sub-task files
├── handlers/
│   └── main.yml          # Handlers (e.g., reboot, service restart)
├── templates/            # Jinja2 templates (.j2) if needed
├── files/                # Static files to copy to managed hosts
├── meta/
│   └── main.yml          # Role metadata: author, dependencies, Galaxy tags
└── molecule/             # (optional) Molecule test scenarios
    └── default/
        ├── molecule.yml
        ├── converge.yml
        └── verify.yml
```

If directories above do not yet exist, create them as needed when implementing features.

---

## Core Functionality

The role is expected to:

1. **Update package lists** — run `apt-get update` on Proxmox hosts.
2. **Upgrade packages** — perform a safe dist-upgrade (`apt-get dist-upgrade`).
3. **Handle Proxmox-specific packages** — avoid inadvertently removing Proxmox enterprise packages or subscription-related components.
4. **Optionally reboot** — reboot nodes when a kernel update is detected (controlled by a variable).
5. **Support both hosts and VMs/containers** — the role may target Proxmox host nodes and/or guest VMs/LXC containers via inventory groups.

---

## Key Variables (defaults/main.yml)

When implementing, define sensible defaults here. Expected variables include:

| Variable | Default | Description |
|---|---|---|
| `proxmox_update_reboot` | `false` | Reboot host after update if kernel changed |
| `proxmox_update_reboot_timeout` | `600` | Seconds to wait for host to come back after reboot |
| `proxmox_update_upgrade_type` | `dist` | apt upgrade type: `safe`, `full`, or `dist` |
| `proxmox_update_autoremove` | `true` | Run `apt autoremove` after upgrade |
| `proxmox_update_cache_valid_time` | `3600` | Seconds before apt cache is considered stale |

---

## Development Conventions

### YAML Style
- Use **2-space indentation** for all YAML files.
- Always quote strings that contain special characters or could be interpreted as other types (e.g., booleans, numbers).
- Use the `| ` block scalar for multiline shell commands in `shell`/`command` tasks.
- Prefer the `ansible.builtin.*` FQCN (Fully Qualified Collection Name) for all built-in modules (e.g., `ansible.builtin.apt`, `ansible.builtin.reboot`).

### Task Structure
- Keep `tasks/main.yml` as a clean orchestrator using `ansible.builtin.include_tasks` to pull in logical sub-files (e.g., `update.yml`, `reboot.yml`).
- Every task must have a descriptive `name:` field written as a sentence (e.g., `"Update apt package cache"`).
- Use `become: true` at the role level or task level where root privileges are required; do not hardcode `sudo`.
- Use `tags:` on tasks to allow selective execution (e.g., `tags: [update, proxmox]`).

### Handlers
- Define a `Reboot Proxmox host` handler in `handlers/main.yml` triggered by tasks that install kernel updates.
- Use `ansible.builtin.reboot` with `reboot_timeout` set from a variable.
- Handlers should be idempotent and only triggered when actually needed.

### Idempotency
- All tasks must be **idempotent** — running the role multiple times must produce the same result.
- Avoid `command`/`shell` tasks unless absolutely necessary; prefer purpose-built modules (`ansible.builtin.apt`, `ansible.builtin.package`, etc.).
- When using `shell` or `command`, always add `changed_when` and `failed_when` conditions.

### Variable Precedence
- User-facing variables belong in `defaults/main.yml` (lowest precedence — easily overridden).
- Internal constants belong in `vars/main.yml` (higher precedence — not meant for users).
- Never hardcode values that should be configurable in tasks; always reference a variable.

---

## Testing

### Molecule (preferred)
Use [Molecule](https://molecule.readthedocs.io/) with the Docker driver for local testing:

```bash
# Install test dependencies
pip install molecule molecule-docker ansible-lint

# Run default scenario
molecule test

# Run only convergence (faster during development)
molecule converge

# Run linting only
molecule lint
```

### Ansible Lint
All YAML/Ansible files must pass `ansible-lint` with no violations:

```bash
ansible-lint
```

Fix any warnings before committing. The `.ansible-lint` config file (if present) defines rule exclusions.

### Manual Testing
To test against a real Proxmox host:

```bash
ansible-playbook -i inventory/hosts.yml playbook.yml --check   # dry-run
ansible-playbook -i inventory/hosts.yml playbook.yml           # live run
```

---

## Git Workflow

- The **default branch** is `main`.
- Feature branches use the format `feature/<short-description>` or `fix/<short-description>`.
- All changes go through pull requests; do not push directly to `main`.
- Commit messages should be imperative, concise, and descriptive (e.g., `Add apt cache update task`, `Fix reboot handler variable reference`).

---

## Proxmox-Specific Notes

- Proxmox VE runs on **Debian** (currently Debian 12 Bookworm for PVE 8.x). All `apt` operations should be compatible with Debian.
- Proxmox has two apt repositories: the **enterprise** repo (requires a subscription) and the **no-subscription** (community) repo. Be careful not to break repository configuration during updates.
- Avoid upgrading `pve-kernel-*` packages without testing — kernel mismatches can prevent VMs from starting.
- The Proxmox `pveversion` command returns the current PVE version and can be used to verify the host type.
- Use `ansible_distribution == "Debian"` and check for Proxmox-specific facts (e.g., presence of `pveversion`) to guard tasks.

---

## AI Assistant Guidelines

- **Read before editing:** Always read existing files before making changes.
- **No unnecessary files:** Do not create files outside the standard Ansible role structure unless there is a clear reason.
- **Stay idempotent:** Every change must preserve the idempotency guarantee.
- **Minimal changes:** Make only the changes requested or clearly required; do not refactor surrounding code.
- **Test awareness:** When adding tasks, consider how they will behave in a Molecule test and whether a `changed_when` condition is needed.
- **Variable discipline:** New configurable values go in `defaults/main.yml` with a comment explaining the variable.
- **No hardcoded credentials:** Never add passwords, tokens, or API keys to any file.
