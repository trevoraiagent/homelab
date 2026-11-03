# Change Log

Newest first. Each entry: what changed, why, how verified, rollback.

## 2026-11-02 - Non-root admin account with 2FA on pve01

**Change:** Created a named administrator account `trevor@pve` (Proxmox VE authentication server realm), granted the Administrator role at path `/`, and enrolled TOTP two-factor authentication using an authenticator app.

**Why:** Day-to-day administration should not use the root account. A named account gives an audit trail, and 2FA protects the web UI against stolen or guessed passwords. `root@pam` is kept as a break-glass fallback.

**Details:**
- Account created under Datacenter > Permissions > Users, username in lowercase because usernames are case-sensitive.
- Role assigned under Datacenter > Permissions with path `/` and role Administrator.
- TOTP enrolled under Datacenter > Permissions > Two Factor.
- Credentials stored in a password manager (Bitwarden), organized in a Homelab folder. Nothing sensitive is stored in this repo.

**Verification:** Logged out and back in as `trevor@pve` with password plus authenticator code. Login succeeded and the dashboard loaded.

**Rollback:** Remove the TOTP entry or delete the user under Datacenter > Permissions. Log in as `root@pam` if the new account is locked out.

## 2026-11-02 - Repository switch and first system update on pve01

**Change:** Disabled the enterprise APT repositories, enabled the No-Subscription repository, and applied all pending updates. Rebooted to load the new kernel.

**Why:** The enterprise repos (pve-enterprise, ceph-squid enterprise) require a paid subscription and returned a warning on a fresh install. No-Subscription is the standard repo for homelab use. Updating right after install picks up security fixes before any workloads exist.

**Details:**
- Disabled `pve-enterprise` and `ceph-squid` enterprise repos (Ceph is not in use).
- Added the `pve-no-subscription` repository via Node > Updates > Repositories.
- Ran Refresh: completed with TASK OK, packages fetched from pve-no-subscription.
- Ran Upgrade: 116 Debian packages pending, including a security update to rsync (33 CVE fixes). Dismissed the apt-listchanges pager and kept existing config files.
- Upgrade finished with "Your System is up-to-date" and a kernel update notice, followed by a reboot.
- The "No valid subscription" popup at login is expected without a subscription and was left as is.

**Verification:** After reboot, `uname -r && pveversion` returned kernel 7.0.14-26-pve and pve-manager 9.2.25 (previously kernel 7.0.2-6-pve, pve-manager 9.2.2). The web UI came back at https://192.168.0.145:8006, which also confirms the address holds after a reboot with the DHCP reservation in place.

**Rollback:** Re-enable the enterprise repos and disable no-subscription in the Repositories panel. Package downgrades are not planned. Select the previous kernel from the boot menu if the new one misbehaves.

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
