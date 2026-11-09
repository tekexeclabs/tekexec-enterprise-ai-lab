# ADR 0001: Lab addressing, names, time and IPv6

| Field | Value |
| --- | --- |
| Status | Accepted (amended Day 12) |
| Date | 2026-11-08 (Day 11, Session 3); amended 2026-11-09 (Day 12) |
| Deciders | Infra, Software |
| Scope | TEK EXEC lab VLANs 10 (MGMT), 20 (PLATFORM), 30 (APPS) |

## Guiding constraint

The lab is never reachable from the internet. There is no inbound NAT, no port forwarding on the home router, no public DNS records and no IPv6. The Project 2 demo runs inside the lab and is shared as a recorded video plus documentation. Every decision below must keep this true.

## Context

Today the lab has four hand-configured devices on VLAN 10: SW1 (10.20.10.2), FW1 MGT (10.20.10.3), pi1 (10.20.10.10) and the admin laptop (10.20.10.200). FW1 is the gateway for every VLAN (10.20.10.1, 10.20.20.1, 10.20.30.1). Starting around Week 11–12, a Dell PowerEdge R730 (SRV1) running Proxmox will add virtual machines to VLANs 20 and 30. Before that happens, we need to decide:

1. Who hands out addresses (DHCP).
2. Who answers name lookups (DNS).
3. Where lab clocks get their time (NTP).
4. Whether the lab uses IPv6.

We also want the lab to resemble real enterprise architecture, so the hands-on work transfers to jobs and is recognizable to recruiters and employers.

## Decision 1: Addressing (DHCP)

**Options considered**

- FW1 as DHCP server.
- All static addresses.
- A dedicated lab server as DHCP server, with FW1 relaying requests between VLANs.

**Decision**

- **VLAN 10 (MGMT) is 100% static. It never has a DHCP pool.**
- **VLANs 20 and 30** use a dedicated lab service VM on SRV1 as the DHCP server: **svc1, at 10.20.20.10 on VLAN 20 (PLATFORM)**.
  - It serves VLAN 20 directly.
  - FW1 relays DHCP requests from VLAN 30 to it.
  - Infrastructure and servers keep static addresses. Only the pool range is dynamic.

**Address ranges (each /24)**

| Range | Use |
| --- | --- |
| .1 | FW1 gateway subinterface |
| .2–.49 | Network gear and infrastructure (on VLAN 10: switch, firewall MGT, jump box, server management; on VLANs 20 and 30: infrastructure services such as svc1) |
| .50–.99 | Static servers and VMs |
| .100–.199 | DHCP pool (VLANs 20 and 30 only) |
| .200–.254 | Admin devices |

This ADR sets the **ranges**. Individual addresses are assigned in the [IP plan](../network/ip-plan.md), which is the single source of truth. Every assignment there must fall inside these ranges.

Existing addresses already fit (SW1 .2, FW1 MGT .3, pi1 .10, laptop .200). Planned SRV1 and service addresses also fit:

| Address | Device | Range |
| --- | --- | --- |
| 10.20.10.20 | SRV1 iDRAC | Infrastructure |
| 10.20.10.21 | SRV1 Proxmox host management | Infrastructure |
| 10.20.20.10 | svc1 (DNS/DHCP service VM) | Infrastructure |
| 10.20.20.50 | mon1 (monitoring VM; moved from 10.20.20.10 in IP plan v1) | Servers/VMs |

**Why**

A central DHCP/DNS server with relay is the common enterprise pattern. It keeps address and name records in one place, and it uses SRV1 as a core part of the architecture. svc1 sits in the infrastructure range because every other host depends on it, like the switch and the firewall.

**Consequences**

- New FW1 security rules will be needed between zones, so that clients can reach svc1. Those rules will be logged.
- When SRV1 is powered off, VLANs 20 and 30 cannot get new leases. Static hosts are unaffected.
- Must check on device: DHCP relay support on FW1 (PAN-OS 9.1.15-h1, unlicensed) for subinterface ethernet1/2.30.

## Decision 2: Names (DNS)

**Options considered**

- Hosts files on each machine.
- FW1 DNS proxy with static entries.
- A dedicated lab DNS server.

**Decision**

- The internal domain is **`lab.internal`**. svc1 (10.20.20.10) becomes the lab's DNS server.
- It answers for `lab.internal` and forwards outside lookups to 1.1.1.1 and 9.9.9.9.

**Until svc1 exists:** pi1 and the admin laptop use hosts files for lab names, adding each name only once its device is configured.

**Names** (addresses as assigned in the IP plan)

| Name | Address | Status |
| --- | --- | --- |
| fw1-gw.lab.internal | 10.20.10.1 (FW1 VLAN 10 gateway) | In use |
| sw1.lab.internal | 10.20.10.2 | In use |
| fw1.lab.internal | 10.20.10.3 (MGT) | In use |
| pi1.lab.internal | 10.20.10.10 | In use |
| srv1-idrac.lab.internal | 10.20.10.20 | Planned (Week 11) |
| srv1.lab.internal | 10.20.10.21 | Planned (Week 12) |
| svc1.lab.internal | 10.20.20.10 | Planned (Week 13) |
| mon1.lab.internal | 10.20.20.50 | Planned (Phase 2) |

**Why**

- `.internal` is reserved for private use and will never exist on the public internet.
- `.local` is avoided because it belongs to mDNS (Bonjour).
- No real public domain is used, which supports the guiding constraint.
- A dedicated DNS server mirrors enterprise practice, and its configuration can be version-controlled.

**Consequences**

- FW1 will need logged rules for VLANs 10 and 30 to reach svc1 on VLAN 20.
- svc1 needs one outbound rule to the internet, for the DNS application only. This is outbound only and does not make the lab reachable.
- The server software is chosen at Week 13 (ADR 0003). Candidates: Windows Server DNS/DHCP, or Linux tools such as BIND or Unbound with ISC Kea.

## Decision 3: Time (NTP)

**Options considered**

- pi1 as the lab time source.
- Each device syncing from the internet.
- Deferring the decision.

**Decision**

- **pi1 (10.20.10.10) is the lab's time source.**
  - pi1 syncs from the internet through FW1, which existing rule `allow-pi1-updates` already permits for NTP.
  - SW1, FW1, SRV1's iDRAC and the Proxmox host sync from pi1.
- All lab devices use the **America/New_York** time zone.

**Why**

- SW1, FW1 MGT, iDRAC, the Proxmox host and pi1 are all on VLAN 10, so time traffic between them never crosses FW1. No new firewall rule is needed.
- None of them needs internet access for time.
- Matching clocks are essential for correlating logs, which Project 2 depends on.

**Consequences**

- pi1 needs an NTP server package (likely `chrony`). The default `systemd-timesyncd` only receives time. Must check on device.
- pi1 has no battery-backed clock. It needs FW1 up to set its own time after boot.
- If pi1 is off, the other devices drift until it returns.
- VMs on VLANs 20 and 30 will need a logged time rule to reach pi1 when they exist.

## Decision 4: IPv6

**Decision**

- **IPv4-only by design.**
  - No IPv6 on FW1 ethernet1/1 or the lab subinterfaces.
  - IPv6 link-local addresses (fe80::) that operating systems create on their own are acceptable. They never cross FW1.

**Why**

Home routers often hand out IPv6 addresses that can be reachable from the internet. Leaving IPv6 out removes a second, unplanned path in and directly supports the guiding constraint.

**Consequences**

Must check on device that FW1 does not forward IPv6 and that pi1 holds only link-local IPv6 addresses.

## Must check on device (results, Day 12, Mon Nov 9, 2026)

- [x] SW1 clock: no time source; starts at the 1993 default after every boot. Eastern time zone and US daylight-saving rule already configured.
- [x] pi1 time: synchronized, NTP service active, time zone America/New_York, no real-time clock. `chrony` not installed; available from Debian (4.6.1-3+deb13u2).
- [x] pi1 IPv6: link-local address only (fe80::…, scope link); no global IPv6.
- [x] FW1 IPv6: no IPv6 configuration on ethernet1/1 or ethernet1/2 and its subinterfaces.
- [x] FW1 DHCP relay: available on ethernet1/2.30 (listed in Network > DHCP > DHCP Relay). Working relay to be proven in Week 13.

## Revisit when

- svc1 is built (Week 13). Choose the DHCP/DNS software (ADR 0003) and write the new FW1 rules.
- VLAN 20 or 30 hosts need time.
- A lab learning goal requires IPv6 internally. Internet reachability is never a reason to revisit.

## Amendments

| Date | Change |
| --- | --- |
| 2026-11-09 (Day 12) | Assigned concrete addresses to the planned hosts (SRV1 iDRAC .20 and Proxmox host .21 on VLAN 10; svc1 10.20.20.10; mon1 10.20.20.50) so the IP plan and this ADR agree. Moved the monitoring VM from 10.20.20.10 (IP plan v1) to 10.20.20.50, because .10 is in the infrastructure range. Made the IP plan the single source of truth for individual addresses. No decision changed. |

## Related

- [IP plan](../network/ip-plan.md)
- [FW1 security policy](../network/fw1-policy.md)
- [Lab cabling](../network/lab-cabling.md)
