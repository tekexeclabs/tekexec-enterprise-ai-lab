# SRV1 build log: Dell PowerEdge R730

Role: virtualization host (Proxmox VE planned, Week 12)
Status (Day 17): hardware verified and tested; iDRAC hardened and in use; old drives sanitized; new SSDs received; no operating system yet (Proxmox VE next)

## Placement

Desk beside the rack, rated well above the server's maximum weight (about 69 lb). Front intake clear, open space behind for exhaust, no direct sunlight. It does not fit the 20"-deep wall cabinet.

## Rear panel (Dell PowerEdge R730 Owner's Manual, matched to our photos)

| Item | Location (facing the rear) | Our plan |
| --- | --- | --- |
| System ID button | Bottom far left ("i") | Not used |
| iDRAC8 dedicated port | Right of the ID button, labeled iDRAC | SW1 Gi0/5, VLAN 10, 10.20.10.20 (in use since Day 12; iDRAC8 Enterprise license) |
| Serial (9-pin) | Teal background | Not used |
| VGA | Blue background | Local monitor during setup |
| USB 3.0 (2) | Right of VGA | Keyboard; installer stick |
| Ethernet 1-4 | Bottom middle, labeled 1-4 | Port 1: trunk to SW1 Gi0/6 (Week 12); ports 2-4 unused |
| PCIe slots 1-7 | Top row (riser 1: 1-3, riser 2: 4-5, riser 3: 6-7) | Empty |
| PSU 1 and 2 | Bottom right, 750 W each | Redundant power |

## Inside (from photos)

| Component | Finding |
| --- | --- |
| Network | Broadcom 5720 quad-port 1 GbE network daughter card (rNDC) |
| Storage controller | PERC H730 Mini with cache battery (verified Day 14); HBA mode available |
| Drive backplane | 2.5-inch, 8 bays. The carrier stickers said "1.92 TB SATA SSD" but were leftovers: the controller found 2 x 1 TB SAS hard drives (corrected Day 13). All 8 slots usable (slot 1 verified Day 16) |
| Other storage | 100 GB SATA SSD in the slim optical bay, on motherboard SATA (previous owner's boot disk; sanitized Day 17, to be removed) |
| Cooling | 6 hot-swap fans |

## Verified hardware (Days 12-15)

| Item | Finding | How verified |
| --- | --- | --- |
| Processors | 2 x Intel Xeon E5-2683 v4 (32 cores, 64 threads) | iDRAC inventory; Dell diagnostics |
| Memory | 128 GB DDR4-2400 ECC | Dell thorough memory test: 9 of 9 tests passed (Day 15) |
| Power supplies | 2 x 750 W, both healthy, about 154 W idle | iDRAC power readings (Day 14) |
| Firmware | BIOS 2.19.0; iDRAC/Lifecycle Controller 2.86.86.86 (final iDRAC8 release); PERC 25.5.9.0001; all current | Compared with Dell downloads (Day 14) |
| Diagnostics | Dell hardware diagnostics reinstalled (missing); package checked by SHA-256 | Install job completed (Day 15) |

## Management (iDRAC8)

Laptop (10.20.10.200) -> SW1 Gi0/1 -> VLAN 10 -> SW1 Gi0/5 -> SRV1 iDRAC (10.20.10.20, `srv1-idrac.lab.internal`).
Same VLAN, so FW1 is not in this path.

Hardening (Day 14):
- Two named administrator accounts; the default root account disabled.
- Only HTTPS (TLS 1.2) and Redfish enabled; SSH, remote RACADM, SNMP and IPMI over LAN off; IPv6 off.
- Host header check kept on (the iDRAC answers only to its own name).

Support status: iDRAC8 reached end of software maintenance in February 2024; 2.86.86.86 is its last release. Protection relies on isolation (management VLAN only, never reachable from home or the internet) and minimal services.

## Storage

Design: [ADR 0002](../decisions/0002-srv1-storage-and-boot.md). Two Intel D3-S4510 960 GB enterprise SSDs in bays 0 and 1, the PERC in HBA mode, a ZFS mirror in Proxmox, UEFI with Secure Boot.

Old drives: sanitized and queued for certified destruction. See [media sanitization](../inventory/media-sanitization.md).

Lesson (Days 14-16): we first blamed slot 1 for a missing drive. The same drive later failed in another slot, and a known-good drive worked in slot 1. Test a suspect slot with a part you trust, and change one thing at a time.

## Next

- Remove the old drives; certified destruction.
- Update the new SSDs' firmware; install them in bays 0 and 1.
- PERC to HBA mode; BIOS to UEFI; network boot removed.
- Install Proxmox VE (10.20.10.21) via iDRAC virtual media (rehearsed Day 17).
