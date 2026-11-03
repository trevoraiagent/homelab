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
| Proxmox VE | 9.2-1 (pve-manager 9.2.2, kernel 7.0.2-6-pve) | pve01 |
| balenaEtcher | (used for install USB) | Admin workstation |

## Network

| Host | FQDN | IP | Method | Role |
|---|---|---|---|---|
| Router | - | 192.168.0.1 | Static | Gateway, DNS |
| pve01 | pve01.homelab.lan | 192.168.0.145/24 | Static on host; DHCP reservation set on router | Proxmox node, web UI :8006 |

## Accounts and Access

| System | Account | Auth | Notes |
|---|---|---|---|
| Proxmox web UI | root@pam | Password | Admin user and 2FA planned. Credentials go in password manager, never in this repo. |
