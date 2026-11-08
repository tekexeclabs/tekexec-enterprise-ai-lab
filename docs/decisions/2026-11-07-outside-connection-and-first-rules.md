# Decision: outside connection and first firewall rules

Date: Day 10 (Sat Nov 7, 2026)
Status: Accepted and in place
Details: [FW1 outside interface](../network/fw1-outside.md) · [FW1 security policy and NAT](../network/fw1-policy.md)

## Context

The lab needs a path to the internet for updates, time and, later, cloud and AI services. Its only way out is through the home router, which also serves family devices. The lab's network gear is past vendor end of support, so it must never be reachable from the home network or the internet, and the lab must never be able to reach home devices.

## Decision

1. **One outside door.** FW1's ethernet1/1 connects to a home router LAN port as zone `untrust`, with an address by DHCP and no management access on that port.
2. **Deny home first.** The top rule blocks and logs any lab connection to the home network.
3. **Allow narrowly.** Only pi1 may go out, only for six named applications on their default ports.
4. **Log every block.** A final logged deny catches everything else leaving the lab.
5. **One outbound NAT rule** for all lab zones. NAT changes addresses; the security rules decide permission.
6. **Public DNS servers** (1.1.1.1 and 9.9.9.9) for the lab, never the home router.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Plug FW1 into the ISP's box or the router's WAN port | Would put end-of-life equipment directly on the internet |
| Static outside address | Could collide with a family device; DHCP lets the home router manage its own network |
| Allow the whole lab "any" to the internet | Fast, but it teaches nothing and opens far more than is needed |
| Rely on the built-in deny | It blocks, but writes nothing in the traffic log, so blocks are invisible |
| Use the home router for lab DNS | Breaks the rule that the lab never talks to home devices |

## Consequences

- Every lab connection to the internet now has a rule name in FW1's traffic log. That is the raw material for the AI operations assistant (Project 2).
- New lab hosts (Proxmox VMs on PLATFORM and APPS) have no internet until a rule is written for them.
- Lab traffic is double-NATed (FW1, then the home router). That's fine for outbound traffic; nothing inbound is allowed.
- Still open, for the Day 11 design session: DHCP and DNS inside the lab, the lab time source, and IPv6.
