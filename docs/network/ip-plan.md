# Lab IP plan (v2)

| Field | Value |
| --- | --- |
| Version | v2, Day 12 (Mon Nov 9, 2026). Replaces v1 (Day 4, Oct 28, 2026) |
| Owners | Infra, Software |
| Decisions it follows | [ADR 0001: Lab addressing, names, time and IPv6](../decisions/0001-lab-dhcp-dns-time-ipv6.md) |
| Rule | This file is the single source of truth for individual lab addresses. ADR 0001 sets the ranges; this file assigns the addresses. |

## Guiding constraint

The lab is never reachable from the internet. It is IPv4-only, and none of its subnets overlap the home network (`<HOME-SUBNET>`). FW1 ethernet1/1 gets its outside address (`<FW1-OUTSIDE>`) from the home router by DHCP; that address is never published.

## VLANs and subnets

| VLAN | Name | Subnet | Gateway (FW1) | FW1 zone | Addressing | Purpose |
| --- | --- | --- | --- | --- | --- | --- |
| 10 | MGMT | 10.20.10.0/24 | 10.20.10.1 (ethernet1/2.10) | mgmt | Static only, never DHCP | Management of lab devices |
| 20 | PLATFORM | 10.20.20.0/24 | 10.20.20.1 (ethernet1/2.20) | platform | Static + DHCP pool | Shared platform services (DNS/DHCP, monitoring, database, AI gateway) |
| 30 | APPS | 10.20.30.0/24 | 10.20.30.1 (ethernet1/2.30) | apps | Static + DHCP pool (relayed by FW1) | Business applications |
| 999 | UNUSED | none | none | none | none | Native VLAN on the trunk; unused ports. Carries no traffic by design |

## Ranges inside every /24 (from ADR 0001)

| Range | Use | VLAN 10 | VLANs 20 and 30 |
| --- | --- | --- | --- |
| .1 | FW1 gateway subinterface | Yes | Yes |
| .2–.49 | Network gear and infrastructure | Switch, firewall MGT, jump box, server management (iDRAC, hypervisor host) | Infrastructure services (DNS/DHCP service VM) |
| .50–.99 | Static servers and VMs | Not used (servers live on 20 and 30) | Monitoring, database, AI gateway, app servers |
| .100–.199 | DHCP pool | **Never** (VLAN 10 is static only) | Dynamic leases for test and app VMs |
| .200–.254 | Admin devices | Admin laptop | Not used yet |

## Address assignments

Status: **In use** = configured and verified on the device. **Planned** = reserved; not configured yet.

### VLAN 10: MGMT (10.20.10.0/24)

| Address | Name | Device | Range | Status |
| --- | --- | --- | --- | --- |
| 10.20.10.1 | fw1-gw.lab.internal | FW1 ethernet1/2.10 (gateway) | Gateway | In use (Day 8) |
| 10.20.10.2 | sw1.lab.internal | SW1 Vlan10 management | Infrastructure | In use (Day 8) |
| 10.20.10.3 | fw1.lab.internal | FW1 MGT port | Infrastructure | In use (Day 8) |
| 10.20.10.10 | pi1.lab.internal | pi1 (jump box, lab time source) | Infrastructure | In use (Day 4) |
| 10.20.10.20 | srv1-idrac.lab.internal | SRV1 iDRAC8 (out-of-band management) | Infrastructure | Planned (Week 11) |
| 10.20.10.21 | srv1.lab.internal | SRV1 Proxmox VE host management | Infrastructure | Planned (Week 12) |
| 10.20.10.200 | (none) | Admin laptop, USB Ethernet adapter (no gateway, no DNS) | Admin | In use (Day 4) |

### VLAN 20: PLATFORM (10.20.20.0/24)

| Address | Name | Device | Range | Status |
| --- | --- | --- | --- | --- |
| 10.20.20.1 | (none) | FW1 ethernet1/2.20 (gateway) | Gateway | In use (Day 8) |
| 10.20.20.10 | svc1.lab.internal | Lab service VM: DNS for `lab.internal`, DHCP for VLANs 20 and 30 | Infrastructure | Planned (Week 13) |
| 10.20.20.50 | mon1.lab.internal | Monitoring and logging VM (Prometheus, Grafana, Loki) | Servers/VMs | Planned (Phase 2) |
| 10.20.20.100–.199 | (dynamic) | DHCP pool | Pool | Planned (Week 13) |

### VLAN 30: APPS (10.20.30.0/24)

| Address | Name | Device | Range | Status |
| --- | --- | --- | --- | --- |
| 10.20.30.1 | (none) | FW1 ethernet1/2.30 (gateway; DHCP relay to 10.20.20.10) | Gateway | In use (Day 8); relay planned (Week 13) |
| 10.20.30.100–.199 | (dynamic) | DHCP pool, served by svc1 through FW1's relay | Pool | Planned (Week 13) |

Later platform and app addresses (database, AI gateway, the four applications) are assigned from .50–.99 when each is built, and added here first.

## Notes

- **SRV1 has two management addresses.** iDRAC (.20) is the server's built-in management chip and works even when the server is off. The Proxmox host (.21) is the operating system's management address. Both stay on VLAN 10. The Proxmox host gets no address on VLANs 20 or 30; it only bridges VM traffic onto them.
- **FW1 has two VLAN 10 addresses.** .1 is the gateway the lab uses to leave its floor. .3 is the MGT port used to administer FW1.
- **Time:** SW1, FW1 and SRV1 sync from pi1 (10.20.10.10), per ADR 0001.

## Hosts files (until svc1 provides DNS)

Add only entries for devices that are **In use**. Add the SRV1 entries on the day each address is configured.

pi1 (`/etc/hosts`) and the admin laptop (`C:\Windows\System32\drivers\etc\hosts`, edited as administrator):

```
10.20.10.2    sw1.lab.internal    sw1
10.20.10.3    fw1.lab.internal    fw1
10.20.10.10   pi1.lab.internal    pi1
```

Added in Week 11 and 12 when configured:

```
10.20.10.20   srv1-idrac.lab.internal    srv1-idrac
10.20.10.21   srv1.lab.internal          srv1
```

## Overlap check

Every lab subnet (10.20.10.0/24, 10.20.20.0/24, 10.20.30.0/24) was checked against `<HOME-SUBNET>` on Day 4: no overlap. Re-check whenever a subnet is added.

## Change history

| Version | Date | Change |
| --- | --- | --- |
| v1 | Day 4, Oct 28, 2026 | VLANs 10, 20, 30 and 999; gateways at .1 on FW1; first monitoring VM proposed at 10.20.20.10 |
| v2 | Day 12, Nov 9, 2026 | Ranges aligned with ADR 0001. Monitoring VM moved from .10 to **10.20.20.50** (.10 is in the infrastructure range and now belongs to the service VM, svc1). SRV1 iDRAC (10.20.10.20) and Proxmox host (10.20.10.21) reserved. Names added for every address |
