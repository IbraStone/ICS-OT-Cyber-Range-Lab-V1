# Phase 1 — Bare-Metal ESXi Provisioning

## 1. Verified installation media

1. Download the ESXi installer ISO from an official vendor portal using enterprise credentials, matching your hardware compatibility list (HCL).
2. Verify integrity before use:
   ```powershell
   Get-FileHash .\VMware-VMvisor-Installer-XXXX.iso -Algorithm SHA256
   ```
   Compare against the vendor-published checksum; discard and re-download on mismatch.
3. Flash the ISO to a clean 16 GB+ USB drive with [Rufus](https://rufus.ie) in **ISO Image mode** (Partition scheme: MBR or GPT to match the target motherboard firmware; file system: FAT32).

## 2. Temporary staging network

To avoid DHCP/IP conflicts with your home/production network during install, stage the target server behind an isolated intermediary host:

- Disable Wi-Fi on the intermediary machine (single-homed only).
- NIC-A: uplinked to your home router (internet/DHCP client).
- NIC-B (USB-Ethernet): runs a scoped local DHCP service (e.g. `10.0.0.100`–`10.0.0.140`, mask `255.255.255.0`, gateway `10.0.0.1`) and connects directly to the target server's management NIC (`vmnic0`) during install only.

## 3. Install ESXi

1. Boot the target server from the USB installer (adjust boot order in BIOS/UEFI as needed).
2. Accept the EULA, select the install target disk, set language, and set a strong root password.
3. Complete the install, remove the USB media, and reboot.

## 4. Hardcode management networking

From the DCUI (`F2` after boot):

| Parameter | Value |
|---|---|
| Static IPv4 Address | `10.0.0.150` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `10.0.0.1` |

Set DNS, apply, restart management networking, then move the host's uplink from the staging intermediary back onto the permanent core network.

**Verification:** `https://10.0.0.150` loads the ESXi host client over the permanent network.
