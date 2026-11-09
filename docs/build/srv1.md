# SRV1 build log: Dell PowerEdge R730

Role: virtualization host (Proxmox VE planned, Week 12)
Status: Day 12, pre-flight inspection complete; not powered, not cabled

## Placement

Desk beside the rack, rated well above the server's maximum weight (about 69 lb). Front intake clear, open space behind for exhaust, no direct sunlight. It does not fit the 20"-deep wall cabinet.

## Rear panel (Dell PowerEdge R730 Owner's Manual, matched to our photos)

| Item | Location (facing the rear) | Our plan |
| --- | --- | --- |
| System ID button | Bottom far left ("i") | Not used |
| iDRAC8 dedicated port | Right of the ID button, labeled iDRAC | SW1 Gi0/5, VLAN 10, 10.20.10.20 (planned; Enterprise license to be verified) |
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
| Storage controller | Integrated Mini-PERC with cache battery; exact model to be confirmed in System Setup |
| Drive backplane | 2.5-inch, 8 bays; 2 x 1.92 TB SATA SSD in bays 0-1 |
| Cooling | 6 hot-swap fans |

## Planned connection (not made yet)

Laptop (10.20.10.200) -> SW1 Gi0/1 -> VLAN 10 -> SW1 Gi0/5 -> SRV1 iDRAC (10.20.10.20).
Same VLAN, so FW1 is not in this path. Any outbound attempt by iDRAC is denied and logged by FW1.
