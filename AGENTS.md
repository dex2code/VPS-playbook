# AGENTS.md — VPS Ansible Playbooks

## Environment
```bash
source .venv/bin/activate    # REQUIRED before every ansible command
ansible --version            # validates setup (ansible-core 2.22+)
```

All remote hosts run **Debian**. Package manager is `apt`, Python interpreter `/usr/bin/python3`.

## Default behavior
- `ansible.cfg` sets `become = false` — every task doesn't escalate to root by default. Override per-playbook with `become: true`.
- Inventory path is relative: `inventory/hosts.ini` (not the Ansible default).
- Host key checking is **enabled** in config (`host_key_checking = true`). Do not disable it; on first connect to a new host accept the key manually (e.g. plain `ssh`) rather than turning checking off.
- Inventory role groups are named with the `_grp` suffix (`xray_grp`, `ingress_grp`, ...). The reserved built-in `all` stays canonical: all hosts are defined in `[all]`, fleet-wide playbooks target bare `all`, and global defaults stay in `group_vars/all.yml`.

## Key files
| File | Purpose |
|---|---|
| `inventory/hosts.ini` | Hosts grouped by role. Replace placeholder IPs + SSH key path. |
| `inventory/group_vars/all.yml` | Global Debian defaults (interpreter, locale, pkg manager). |
| `inventory/host_vars/<hostname>.yml` | Per-host overrides — filename must match inventory alias, not DNS name. |

## Common commands
```bash
source .venv/bin/activate                              # activate venv first!
ansible-playbook playbooks/ping.yml                    # connectivity + diagnostics on all hosts
ansible-playbook playbooks/apt-upgrade.yml             # maintenance: apt upgrade + reboot (serial: 1)
ansible-playbook playbooks/xray-upgrade.yml            # upgrade xray-core (inventory group [xray_grp]; release downloaded on controller)
ansible-playbook playbooks/ping.yml --limit vortex     # single host (see inventory/hosts.ini for aliases)
ansible-playbook playbooks/ping.yml -i <other.ini>     # alternate inventory
```

## Gotchas
- `.venv` is gitignored. Any new agent must recreate it: `python3 -m venv .venv && pip install -r requirements.txt`.
- Don't hardcode secrets in inventory — use `group_vars/`, `host_vars/`, or Ansible Vault (`ansible-vault encrypt_string 'secret' --name var_name`).
- Inline connection parameters in `inventory/hosts.ini` (`ansible_host`, `ansible_port`, `ansible_user`, `ansible_ssh_private_key_file`) are the **project convention — keep them inline per host, do not move them to `group_vars/`/`host_vars/`**. Use `host_vars/<hostname>.yml` only for non-connection settings (app config, secrets via Vault, etc.).
- File transfers (`copy`/`unarchive`/`synchronize`) used to hang on these hosts over the default sftp. `transfer_method = piped` in `ansible.cfg` fixes it — do not remove. If a transfer stalls again, check this setting first, not the network.
