# FW1 build log: Palo Alto PA-220

Role: firewall and router between lab zones; the lab's only path to the outside
First power-on: Day 6 (Fri Oct 30, 2026)
Status (Day 17): running under supervision; lab zones built Day 8, trunk to SW1 Day 9, outside connection and first logged rules Day 10

## Current configuration

- Zones and VLAN subinterfaces: [lab cabling](../network/lab-cabling.md) and [IP plan](../network/ip-plan.md)
- Outside interface: [FW1 outside interface](../network/fw1-outside.md)
- Security rules and NAT: [FW1 security policy](../network/fw1-policy.md)
- Known-good configuration snapshot: `day10-after.xml`

## As found

| Item | Value |
| --- | --- |
| PAN-OS | 9.1.15-h1 (9.1 is end of life; no support contract, so no upgrades) |
| Content | App-ID 8635; no threat, antivirus, URL or WildFire content |
| Licenses | None |
| Configuration | Factory default: hostname PA-220, MGT 192.168.1.1/24, only rule1 plus default rules |
| Health | All environmental alarms False; 39–44 °C; 12 V input rail reading 12.00 V |
| Interfaces | Only ethernet1/1 and 1/2 configured, both down; port LEDs lit without link (cosmetic) |

## Power

Runs on a third-party 12 V DC, 3 A, center-positive adapter with a threaded locking plug, bought for FW1 (Palo Alto's own part is PAN-PA-220-PWR-AC). Before use we checked it against FW1's label and Palo Alto's specifications: same voltage, more current than the 1.75 A FW1 needs, matching polarity and plug. It has run FW1 since Day 6 with no power, temperature or connectivity problems. Decision (Nov 23, 2026): keep this adapter. FW1 is used under supervision and shut down cleanly after each session.

## Access

Console: RJ-45 CONSOLE port, 9600 baud, 8N1, no flow control. Boot to a usable login takes about 25 minutes (wait for STAT green).
Default admin password changed and committed. Shut down with `request shutdown system`, then remove power.

## Lessons

- The login prompt appears before the firewall is ready. STAT green is the real signal.
- On PAN-OS, a change isn't saved until it's committed. `show config diff` shows what's still pending.
