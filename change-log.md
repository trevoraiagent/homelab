# Change Log

Newest first. Each entry: what changed, why, how verified, rollback.

## 2026-11-02 - DHCP reservation for pve01

**Change:** Created a DHCP address reservation on the router binding 192.168.0.145 to pve01's network interface (vmbr0 MAC).

**Why:** pve01 uses a static IP configured on the host. The reservation stops the router from handing .145 to another device and causing an IP conflict.

**Details:**
- pve01 did not appear in the router's connected-devices list because it never requests a lease, so the MAC was entered manually.
- MAC address read from the Proxmox shell with `ip -br link show vmbr0`.
- Hostname also confirmed with `hostname -f`, which returned `pve01.homelab.lan`.

**Verification:** Reservation saved in the router. Re-check after the next reboot that pve01 still answers at 192.168.0.145.

**Rollback:** Delete the reservation entry in the router's DHCP settings. The host keeps working on its static IP.

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
2. ~~Create DHCP reservation for 192.168.0.145 on the router.~~ Done 2026-11-02 (see entry above).
