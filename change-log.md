# Change Log

Newest first. Each entry: what changed, why, how verified, rollback.

## 2026-11-02 - Initial Proxmox VE install on pve01

**Change:** Installed Proxmox VE 9.2-1 on a used Dell OptiPlex 5070 Micro.

**Details:**
- Created install media with balenaEtcher (Proxmox VE 9.2-1 ISO to USB).
- Booted the OptiPlex from USB. The Bluetooth keyboard was not recognized at the boot menu, so a wired USB keyboard was used instead.
- Installed to /dev/nvme0n1 using ext4.
- Hostname: pve01.homelab.lan
- Network: 192.168.0.145/24, gateway 192.168.0.1, DNS 192.168.0.1
- Time zone: America/Chicago
- Alert email configured for system notifications.

**Verification:** System rebooted cleanly to the login prompt. Web UI reachable at https://192.168.0.145:8006, logged in as root@pam. Node Summary (uptime about 11 min) confirmed: 6 x Intel Core i5-9500T @ 2.20 GHz (1 socket), 15.40 GiB RAM, kernel Linux 7.0.2-6-pve, EFI boot mode, pve-manager 9.2.2, root filesystem 67.73 GiB on the default LVM layout, 8 GiB swap. Idle load: 0.79% CPU, 10.53% RAM. Evidence captured: Etcher flash, network config, install summary, boot login, and node Summary screenshots.

**Rollback:** Reinstall from USB media (no data on host yet).

**Follow-ups:**
1. ~~Confirm the full FQDN with `hostname -f`.~~ Done 2026-11-02: shell returned `pve01.homelab.lan`.
2. Create DHCP reservation for 192.168.0.145 on the router.
