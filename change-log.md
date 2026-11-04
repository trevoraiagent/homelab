# Change Log

Newest first. Each entry: what changed, why, how verified, rollback.

## 2026-11-02 - Scheduled backup job for pihole01

**Change:** Created a Proxmox backup job for CT 100 (pihole01) on the `local` storage, scheduled daily at 02:00, snapshot mode, ZSTD compression, keeping the last 3 backups. Ran it once manually to test.

**Why:** Snapshots protect against configuration mistakes but live with the container. A backup is a separate file that can restore the container after deletion or corruption, and a daily schedule means recovery never loses more than a day of changes.

**Details:**
- Created under Datacenter > Backup > Add. Selection: CT 100 only.
- Retention keep-last 3 limits disk use on the single 256 GB drive.
- Limitation: the backup target is the same physical disk as the host. This does not protect against drive failure. An off-host copy (USB drive or another machine) is still needed.

**Verification:** Used "Run now"; the task list showed the backup task with status OK.

**Rollback:** Delete the job under Datacenter > Backup. Existing backup files can be removed under local > Backups.

**Follow-ups:**
1. Confirm the backup file appears under local (pve01) > Backups and record its size.
2. Practice a restore into a new container ID to prove the backup works. An untested backup is not a backup.
3. Add an off-host backup target when hardware allows.

## 2026-11-02 - Snapshot of pihole01 after clean Pi-hole install

**Change:** Took a Proxmox snapshot of CT 100 named `clean-pihole-install`, without RAM state.

**Why:** A known-good restore point before testing Pi-hole with real devices. If a setting or update breaks the container, it can be rolled back in a few clicks instead of rebuilt.

**Details:**
- Taken from CT 100 > Snapshots > Take Snapshot. State at the time: Pi-hole v6.5 / Web v6.6.1 / FTL v6.8 installed, dashboard reachable, no clients using it.
- A snapshot lives on the same disk as the container, so it protects against configuration mistakes, not against disk failure. A separate backup job is the next task.

**Verification:** Task completed with TASK OK.

**Rollback:** Delete the snapshot from the same Snapshots panel. To use it, select it and choose Rollback (this discards changes made after the snapshot).

## 2026-11-02 - Pi-hole installed and tested on pihole01

**Change:** Installed Pi-hole, a DNS-level ad and tracker blocker, inside CT 100 using the official installer. No client device and no router DNS setting was changed, so nothing else on the network uses it yet.

**Why:** Hands-on practice with DNS administration, and a safe first service to run on the lab. Keeping it unused until tested means a mistake cannot affect other devices.

**Details:**
- Confirmed the container had internet and name resolution first (`ping deb.debian.org` returned replies), after installing `curl`.
- Ran the Pi-hole installer; it finished with "Installation Complete". Used the default blocklist and the "Show everything" query-logging privacy mode.
- Installed versions (all current at install time): Core v6.5, Web v6.6.1, FTL v6.8.
- Entering the container with `pct enter` gives a shell without `/usr/local/bin` on its PATH, so `pihole` returned "command not found" until it was run by full path (`/usr/local/bin/pihole`).
- Could not log in to the container console with the root password set at creation (typed blind, so a typo was likely). Entered the container from the host with `pct enter 100` and reset the password with `passwd`.

**Verification:** From the admin PC, queried Pi-hole directly without changing any settings. `nslookup example.com 192.168.0.146` returned real addresses. `nslookup doubleclick.net 192.168.0.146` returned `0.0.0.0`, showing the ad domain is blocked. The server identified itself as `pi.hole` at 192.168.0.146.

**Rollback:** No device depends on it. Remove with `pihole uninstall` inside the container, or stop and remove CT 100.

**Follow-ups:**
1. ~~Set the dashboard password and confirm the dashboard.~~ Done 2026-11-02: admin password set and the dashboard loads at http://192.168.0.146/admin.
2. Confirm the upstream DNS provider and blocklist settings in the dashboard. (Version recorded above.)
3. Point one test device at 192.168.0.146 manually and watch the query log.
4. Snapshot the container, then add it to a backup job.
5. Decide whether to use Pi-hole for the whole network. If the router uses it, a secondary DNS fallback matters, because everyone's internet depends on this container being up.

## 2026-11-02 - First container: pihole01 (LXC) on pve01

**Change:** Created an unprivileged LXC container, CT 100 `pihole01`, as the host for a future Pi-hole DNS filter. Pi-hole itself is not installed yet.

**Why:** First real workload on the host. A container is much lighter than a full VM for a single network service. No other device uses it yet, so it can be tested without affecting the rest of the network.

**Details:**
- Downloaded the Debian 13 standard container template (debian-13-standard_13.6-1) to local storage.
- Created CT 100: hostname `pihole01`, unprivileged, 1 core, 512 MiB RAM, 512 MiB swap, 8 GiB root disk on local-lvm.
- Network: bridge vmbr0, static 192.168.0.146/24, gateway 192.168.0.1, DNS left at the host default.
- Set to start after creation. Root password stored in the password manager.
- Caught a blank IPv4 address field while reviewing the Confirm page and corrected it before creating the container.

**Verification:** Task log shows CT 100 Create and CT 100 Start both OK. The container Summary shows status running, IP 192.168.0.146, 0% CPU, and about 19 MiB of 512 MiB RAM in use.

**Rollback:** Stop and remove CT 100 (More > Remove). Nothing depends on it yet. The router reservation for 192.168.0.146 would also be deleted.

**Follow-ups:**
1. ~~Create a DHCP reservation for 192.168.0.146 using the container's MAC (container > Network).~~ Done 2026-11-02: reservation saved on the router.
2. Install Pi-hole and test it from a single device before changing any router DNS settings.
3. Take a snapshot after setup, then add the container to a backup job.

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
- Ran Upgrade: 116 Debian-origin packages pending (plus Proxmox packages, including the kernel), including a security update to rsync (33 CVE fixes). Dismissed the apt-listchanges pager and kept existing config files.
- Upgrade finished with "Your System is up-to-date" and a kernel update notice, followed by a reboot.
- The "No valid subscription" popup at login is expected without a subscription and was left as is. The Summary page also shows "Non production-ready repository enabled", which is expected while using the no-subscription repository.

**Verification:** After reboot, `uname -r && pveversion` returned kernel 7.0.14-26-pve and pve-manager 9.2.25 (previously kernel 7.0.2-6-pve, pve-manager 9.2.2). The web UI came back at https://192.168.0.145:8006, which confirms the static address survives a reboot. The DHCP reservation itself has not been independently tested.

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
