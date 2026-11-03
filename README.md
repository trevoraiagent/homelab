# Homelab

Personal homelab built to practice IT support, systems administration, security, and automation skills. Everything here is documented the way a production environment would be: inventory, network layout, change log, and runbooks.

## Summary

I built this lab on a used Dell OptiPlex 5070 Micro running Proxmox VE. Day one: installed Proxmox, set a static IP and hostname, reserved the address on the router, switched from the paid enterprise repos to the no-subscription repo, and applied all updates including a kernel and security fixes. Every change is recorded in the change log with verification and rollback steps, and the install is written up as a runbook. Next up: a non-root admin account with 2FA, backups, and my first VMs.

## Environment

| Item | Detail |
|---|---|
| Hypervisor | Proxmox VE 9.2-1 |
| Node | pve01 (pve01.homelab.lan) |
| Hardware | Dell OptiPlex 5070 Micro, i5-9500T, 16 GB RAM, 256 GB NVMe |
| Management | https://192.168.0.145:8006 |
| Network | 192.168.0.0/24, gateway/DNS 192.168.0.1 |

## Network Diagram

```mermaid
flowchart LR
    ISP[(Internet)] --> RTR[Router<br/>192.168.0.1<br/>Gateway / DHCP / DNS]
    RTR --> PVE[pve01<br/>192.168.0.145<br/>Proxmox VE 9.2-1]
    RTR --> LAN[Other LAN clients]
    PVE --> VMS[(Future VMs / containers)]
```

## Repo Contents

- [inventory.md](inventory.md) - hardware, software, and network assets
- [change-log.md](change-log.md) - dated record of every change
- [runbook-proxmox-install.md](runbook-proxmox-install.md) - how pve01 was built

## Roadmap

- [x] Reserve 192.168.0.145 in router DHCP
- [x] Switch to no-subscription repos, run first update
- [x] Create a non-root admin user with 2FA
- [ ] Backup job and storage layout
- [ ] First VM/container, then document as a runbook
