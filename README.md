# Proxmox_VMs_Update

Ansible role to update Proxmox VE hypervisor nodes, LXC containers, and VM guests.

## What it does

| Target | Method | Requires |
|---|---|---|
| Proxmox host nodes | `apt dist-upgrade` via SSH | SSH access to Proxmox host as root |
| LXC containers | `pct exec <vmid> -- apt-get upgrade` | Running on the Proxmox host |
| VM guests | `apt dist-upgrade` via SSH | SSH access to each VM guest |

LXC containers are discovered automatically via `pct list` — no inventory entry needed.

## Requirements

- Ansible >= 2.12
- Proxmox VE 7.x or 8.x (Debian Bullseye / Bookworm)
- SSH access to Proxmox nodes as `root`
- VM guests must be reachable via SSH and have `apt` (Debian/Ubuntu)

## Quick start

```bash
# 1. Copy the example inventory and edit it
cp inventory/hosts.yml.example inventory/hosts.yml
$EDITOR inventory/hosts.yml

# 2. Dry-run (no changes applied)
ansible-playbook -i inventory/hosts.yml playbook.yml --check

# 3. Apply updates
ansible-playbook -i inventory/hosts.yml playbook.yml
```

## Role variables

| Variable | Default | Description |
|---|---|---|
| `proxmox_update_reboot` | `false` | Reboot after upgrade if `/var/run/reboot-required` exists |
| `proxmox_update_reboot_timeout` | `600` | Seconds to wait for host to come back after reboot |
| `proxmox_update_upgrade_type` | `dist` | apt upgrade type: `safe`, `full`, or `dist` |
| `proxmox_update_autoremove` | `true` | Run `apt autoremove` after upgrade |
| `proxmox_update_autoclean` | `true` | Run `apt autoclean` after upgrade |
| `proxmox_update_cache_valid_time` | `0` | Seconds before apt cache is stale (0 = always refresh) |
| `proxmox_update_lxc` | `true` | Update LXC containers via `pct exec` |
| `proxmox_update_skip_stopped_lxc` | `true` | Skip LXC containers that are stopped |
| `proxmox_update_lxc_timeout` | `300` | Per-container apt command timeout in seconds |

Override any variable on the command line:

```bash
ansible-playbook -i inventory/hosts.yml playbook.yml \
  -e proxmox_update_reboot=true \
  -e proxmox_update_upgrade_type=full
```

## Selective execution with tags

```bash
# Update Proxmox hosts only (no LXC, no VMs)
ansible-playbook -i inventory/hosts.yml playbook.yml --tags host

# Update LXC containers only
ansible-playbook -i inventory/hosts.yml playbook.yml --tags lxc

# Update VM guests only
ansible-playbook -i inventory/hosts.yml playbook.yml --tags vms

# Run everything but skip reboots
ansible-playbook -i inventory/hosts.yml playbook.yml --skip-tags reboot
```

## Directory structure

```
Proxmox_VMs_Update/
├── playbook.yml                   # Top-level playbook
├── inventory/
│   └── hosts.yml.example          # Copy → hosts.yml and edit
├── defaults/
│   └── main.yml                   # User-facing role variables
├── vars/
│   └── main.yml                   # Internal role constants
├── tasks/
│   ├── main.yml                   # Task orchestrator
│   ├── update_proxmox_host.yml    # Host apt upgrade tasks
│   ├── update_lxc_containers.yml  # LXC update tasks (via pct exec)
│   └── update_vms.yml             # VM guest apt upgrade tasks
├── handlers/
│   └── main.yml
└── meta/
    └── main.yml
```

## License

Apache 2.0
