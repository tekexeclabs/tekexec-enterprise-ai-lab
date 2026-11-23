# ADR 0002: SRV1 storage and boot design

| Field | Value |
| --- | --- |
| Status | Accepted (amended Day 17) |
| Date | 2026-11-12 (Day 15); amended 2026-11-13 (Day 16) and 2026-11-15 (Day 17) |
| Deciders | Infra, Software |
| Scope | SRV1 (Dell PowerEdge R730): storage controller, drives, boot mode, retired media |

## Context

SRV1 will run Proxmox VE and every lab virtual machine. When this decision was made:

- Storage controller: PERC H730 Mini in RAID mode; HBA mode available.
- Drives: two 1 TB SAS hard drives made in 2013 with non-Dell firmware. One fails Dell diagnostics and logs intermittent faults.
- A 100 GB SATA SSD on the motherboard's SATA port, in the slim optical bay (a previous owner's boot disk).
- Boot mode: legacy BIOS.
- Memory: 128 GB ECC, passed Dell's thorough memory test.

Requirements: survive a single disk failure; detect silent data corruption; support snapshots for safe changes; follow current Proxmox and industry practice; stay within the lab budget.

## Decision

1. **Controller:** switch the PERC H730 Mini to HBA mode and use a ZFS mirror (RAID 1) managed by Proxmox. Pool settings: ashift=12, lz4 compression, monthly scrub, ZFS event alerts, pool kept under 80% full.
2. **Drives:** two used enterprise SATA SSDs, 960 GB class, with power-loss protection (bought: 2 x Intel D3-S4510 960 GB; the seller's SMART reports show health 100% and zero media errors, to be confirmed on our own server before use). Buy only drives with a health report of 90% or more and returns accepted. Update firmware to the vendor's latest before use.
3. **Layout:** the Proxmox operating system and VM storage both live on the ZFS mirror. *(Amended Day 17: the 100 GB SSD is removed from the server and destroyed, not kept as a disabled spare.)*
4. **Boot:** UEFI with Secure Boot enabled. Network boot (PXE) removed from the boot order.
5. **Bays:** the new SSDs go in **bays 0 and 1**. *(Amended Day 16: slot 1 was retested with a known-good drive and works; the earlier fault came from the old drive, not the slot.)*
6. **Retired media:** sanitized under NIST SP 800-88 Rev. 2, then destroyed by a certified service with a certificate of destruction. See [media sanitization](../inventory/media-sanitization.md).

## Alternatives considered

- **PERC hardware RAID 1 + LVM-thin:** a valid enterprise pattern, but no end-to-end checksums, and recovery is tied to this specific card. Proxmox advises against running ZFS on top of a hardware RAID controller with its own cache.
- **480 GB drives:** about half the cost, but too little room under the 80% fill guideline once monitoring, logs and AI data grow.
- **New consumer SSDs:** no power-loss protection.
- **New enterprise SSDs at retail:** 3-5 times the price of used enterprise drives with health reports.
- **100 GB SSD as a separate boot disk:** a single point of failure outside the mirror, with unknown wear.
- **Internal SD card module for boot:** Proxmox writes logs and metrics constantly; SD cards wear out quickly.
- **Legacy BIOS boot:** Proxmox warns against upgrading a ZFS root pool on legacy-BIOS (GRUB) systems; UEFI avoids this and allows Secure Boot.

## Consequences

- **Positive:** the platform survives a single drive failure; ZFS detects and repairs silent corruption; snapshots allow quick rollback; matches current Proxmox practice; Secure Boot blocks unsigned boot code.
- **Negative:** switching to HBA mode required deleting the old virtual disk (done Day 16). ZFS reserves up to 16 GiB of RAM for cache by default. Used drives carry some wear (mitigated by health checks, returns and the mirror).
- **Not a backup:** a mirror protects against disk failure only. Backups go to a separate target (after the Proxmox build), following the 3-2-1 rule.
- **Validation (at the Proxmox install):** Proxmox shows each SSD's real model; SMART health readable; `zpool status` reports ONLINE with no errors; Secure Boot reported as enabled.

## Cost

$300 plus tax for the two SSDs (November 2026, used, from a 99.7%-rated seller with 30-day returns).
