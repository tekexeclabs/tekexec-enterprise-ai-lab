# FW1 ↔ SW1 link plan (Week 7)

Status: planned, not yet configured. Decided Day 7 (Sat Oct 31, 2026).
Builds on: [IP plan v1](ip-plan.md), [SW1 hardening notes](sw1-hardening.md), [FW1 build log](../build/fw1.md)

## Picture

    Home router
        │  Cable 1: untrust, the lab's only way out
    FW1 (PA-220)
        │ ethernet1/2 ══ trunk (VLANs 10, 20, 30) ══ SW1 Gi0/3     Cable 2
        │ MGT ──────── access VLAN 10 ──────────── SW1 Gi0/4     Cable 3
    SW1 → Gi0/1 laptop, Gi0/2 pi1 (VLAN 10)

## Cables

| # | From | To | Purpose |
| --- | --- | --- | --- |
| 1 | FW1 ethernet1/1 | Home router LAN port | Untrust: outbound internet only |
| 2 | FW1 ethernet1/2 | SW1 Gi0/3 | Trunk carrying VLANs 10, 20, 30 |
| 3 | FW1 MGT | SW1 Gi0/4 | Out-of-band management of FW1 |

## FW1

| Interface | Mode | Tag | Address | Zone |
| --- | --- | --- | --- | --- |
| ethernet1/1 | Layer 3 | none | Home-network client (method decided in Week 7) | untrust |
| ethernet1/2 | Layer 3 parent, no address | none | none | none |
| ethernet1/2.10 | Layer 3 subinterface | 10 | 10.20.10.1/24 | mgmt |
| ethernet1/2.20 | Layer 3 subinterface | 20 | 10.20.20.1/24 | platform |
| ethernet1/2.30 | Layer 3 subinterface | 30 | 10.20.30.1/24 | apps |
| MGT | Management | n/a | 10.20.10.3/24, gateway 10.20.10.1 | n/a |

- One zone per VLAN: every flow between zones needs an explicit, logged rule. Rules are written in Week 8.
- Time: time zone US/Eastern, clock set by hand, then NTP once the untrust link works.

## SW1

| Port | Mode | VLANs |
| --- | --- | --- |
| Gi0/3 | Trunk | Allowed 10, 20, 30; native 999 (not allowed, so untagged frames are dropped) |
| Gi0/4 | Access, portfast | 10 |

SW1 default gateway: 10.20.10.1.

## Why these choices

- **One trunk instead of three cables:** adding a VLAN later costs configuration, not cables and ports. Shared 1 Gbps is plenty for this lab.
- **The firewall routes between VLANs, not the switch:** traffic crossing zones goes up the trunk and back down, so FW1's rules see all of it.
- **Native VLAN 999:** untagged traffic can't land in a real zone, which blocks mistakes and VLAN-hopping tricks.
- **Dedicated MGT cable:** a mistake in a traffic rule can't lock us out of the firewall.

## Required before any data cable connects

FW1 ships with a factory virtual wire (`default-vwire`) that bridges ethernet1/1 (zone untrust) and ethernet1/2 (zone trust), plus rule1 allowing trust → untrust. Connected as-is, it would join the lab to the home network. Week 7's first change deletes rule1, both zones, the virtual wire and the two interfaces' settings in a single commit, before any cable goes in.
