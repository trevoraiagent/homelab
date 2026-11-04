# Inventory

Last updated: 2026-11-02

## Hardware

| Asset | Detail | Source | Notes |
|---|---|---|---|
| pve01 | Dell OptiPlex 5070 Micro | Used (eBay) | Hypervisor host |
| CPU | Intel Core i5-9500T @ 2.20 GHz, 6 cores, 1 socket | - | Verified on Proxmox Summary 2026-11-02 |
| RAM | 16 GB (15.40 GiB usable) | - | Verified on Proxmox Summary 2026-11-02 |
| Storage | 256 GB NVMe (/dev/nvme0n1) | - | Single disk, ext4 on LVM, EFI boot |

## Software

| Component | Version | Host |
|---|---|---|
| Proxmox VE | Installed from 9.2-1 ISO; now pve-manager 9.2.25, kernel 7.0.14-26-pve (updated 2026-11-02) | pve01 |
| Pi-hole | Core v6.5, Web v6.6.1, FTL v6.8 (installed 2026-11-02) | pihole01 (CT 100) |
| balenaEtcher | (used for install USB) | Admin workstation |

## Virtual Machines and Containers

| ID | Name | Type | Resources | Address | Purpose | Status |
|---|---|---|---|---|---|---|
| 100 | pihole01 | LXC (unprivileged), Debian 13 | 1 core, 512 MiB RAM, 8 GiB disk | 192.168.0.146/24 (static) | Pi-hole DNS filter | Running, Pi-hole installed; no clients using it yet |

## Network

| Host | FQDN | IP | Method | Role |
|---|---|---|---|---|
| Router | - | 192.168.0.1 | Static | Gateway, DNS |
| pve01 | pve01.homelab.lan | 192.168.0.145/24 | Static on host; DHCP reservation set on router | Proxmox node, web UI :8006 |
| pihole01 | - | 192.168.0.146/24 | Static on container; DHCP reservation set on router | CT 100, future DNS filter |

## Accounts and Access

| System | Account | Auth | Notes |
|---|---|---|---|
| Proxmox web UI | trevor@pve | Password + TOTP | Administrator role. Daily-use account. |
| Proxmox web UI | root@pam | Password | Break-glass fallback only. Credentials in password manager, never in this repo. |
