# Runbook: Install Proxmox VE on pve01

**Target:** Dell OptiPlex 5070 Micro (i5-9500T, 16 GB RAM, 256 GB NVMe)
**Version:** Proxmox VE 9.2-1
**Last run:** 2026-11-02

## Prerequisites

- Proxmox VE ISO downloaded from the official Proxmox site (verify the SHA256 checksum)
- USB drive (data on it will be erased)
- balenaEtcher on a workstation
- **Wired** USB keyboard and a monitor (Bluetooth keyboards do not work at the installer)
- Ethernet connection to the router
- Planned values: hostname, static IP, gateway, DNS

## Steps

1. **Create install media.** Open balenaEtcher, select the ISO, select the USB drive, flash.
2. **Boot from USB.** Power on the OptiPlex, press F12 for the one-time boot menu, choose the USB drive.
3. **Start installer.** Select "Install Proxmox VE (Graphical)", accept the EULA.
4. **Target disk.** Choose /dev/nvme0n1, filesystem ext4.
5. **Locale.** Country, time zone America/Chicago, keyboard layout.
6. **Credentials.** Set a strong root password and an email for alerts. Store the password in a password manager.
7. **Network.**
   - Hostname (FQDN): pve01.homelab.lan
   - IP/CIDR: 192.168.0.145/24
   - Gateway: 192.168.0.1
   - DNS: 192.168.0.1
8. **Review the summary screen and install.** Remove the USB when prompted and reboot.

## Verification

- Console shows the Proxmox login prompt with the correct IP.
- Browse to https://192.168.0.145:8006 (accept the self-signed certificate warning).
- Log in as root, realm Linux PAM.
- Node Summary shows the expected CPU, RAM, and hostname.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Keyboard unresponsive at boot/installer | Bluetooth keyboard, no BT stack pre-OS | Use wired USB keyboard |
| Web UI unreachable | Wrong IP, cable, or firewall | Check console for IP, ping from workstation, use https and port 8006 |

## Post-install checklist

- [ ] DHCP reservation for 192.168.0.145 on the router
- [ ] Switch to the no-subscription repository and run `apt update && apt full-upgrade`
- [ ] Create a non-root admin user and enable 2FA
- [ ] Configure backups
