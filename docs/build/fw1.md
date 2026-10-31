# FW1 build log: Palo Alto PA-220

Role: firewall and router between lab zones (plan Week 7)
First power-on: Day 6 (Fri Oct 30, 2026)

## As found

| Item | Value |
| --- | --- |
| PAN-OS | 9.1.15-h1 (9.1 is end of life; no support contract, so no upgrades) |
| Content | App-ID 8635; no threat, antivirus, URL or WildFire content |
| Licenses | None |
| Configuration | Factory default: hostname PA-220, MGT 192.168.1.1/24, only rule1 plus default rules |
| Health | All environmental alarms False; 39–44 °C; 12.0 V input rail at 12.00 V |
| Interfaces | Only ethernet1/1 and 1/2 configured, both down; port LEDs lit without link (cosmetic) |

## Power

Runs on a third-party 12 V 3 A center-positive adapter with a threaded locking plug (Palo Alto specifies its own adapter, PAN-PA-220-PWR-AC). Rule: supervised use only, unplugged after each session, replaced before 24/7 use.

## Access

Console: RJ-45 CONSOLE port, 9600 baud, 8N1, no flow control. Boot to a usable login takes about 25 minutes (wait for STAT green).
Default admin password changed and committed. Shut down with `request shutdown system`, then remove power.

## Lessons

- The login prompt appears before the firewall is ready. STAT green is the real signal.
- On PAN-OS, a change isn't saved until it's committed. `show config diff` shows what's still pending.
