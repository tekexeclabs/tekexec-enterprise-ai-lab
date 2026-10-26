# Hardware inventory

Last updated: October 26, 2026 (Day 1). Public version: serial numbers
## Active lab equipment

| Tag | Device | Model (from label) | Power input (from label) | Support status | Role |
| --- | --- | --- | --- | --- | --- |
| FW1 | Palo Alto PA-220 | PA-220 | 12 V DC, 1.75 A, external adapter; two inputs | End of sale Jan 31, 2023; end of life Jan 31, 2028; last supported PAN-OS 10.2 | Lab edge firewall; routes and inspects traffic between lab zones |
| SW1 | Cisco Catalyst 2960G | WS-C2960G-24TC-L | AC, 100–240 V | Past Cisco end of support (2017) | The lab's single switch |
| SRV1 | Dell PowerEdge R730 | 8 x 2.5" bay chassis, 2x 750 W power supplies, 2x 1.92 TB SATA SSD | AC, hot-swap supplies | No model-wide end-of-life date; iDRAC8 end of software maintenance Feb 2024 | Proxmox VE host |
| PI1 | Raspberry Pi 4 (4 GB) | CanaKit kit | USB-C | Current | Uptime checks, jump box |

## Shelved spares

| Device | Model | Why Shelved |
| --- | --- | --- |
| Cisco Catalyst 2960G | WS-C2960G-24TC-L | One switch covers the lab |
| Cisco Catalyst 3560G | WS-C3560G-24TS-S | Routing between zones moved to the firewall |
| Cisco ISR 4331 | ISR4331/K9 | Branch routing adds little for an AI and cloud path |

## Rack

Sysracks 9U wall-mount cabinet, 20" usable depth, 133 lb rating. The switch and firewall fit inside; the R730 (about 25.5" deep) sits on a separate sturdy surface.

## Security Note

Every network device here is past or near its manufacturer's end of support. None of it will ever face the internet directly: the lab sits behind the firewall behind the ISP router

## Upcoming Tasks - Verify

- [ ] Actual CPU, memory and drive health, read from iDRAC
- [ ] Firmware and software versions on every device
- [ ] PA-220 license status
- [ ] Measured power draw of each device
