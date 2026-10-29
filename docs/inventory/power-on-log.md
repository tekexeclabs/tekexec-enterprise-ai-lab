# Power-on log

Rules: one device at a time, read-only before any change, every change logged with its risk and recovery plan, nothing connected to the home network.
Power: GE surge protector (Type 3 SPD, VPR 400 V, 15 A), plugged directly into a wall outlet.
Private details (Service Tag, serial numbers, MAC addresses, home network addresses, usernames, passwords, SSH key fingerprints, previous owners' configuration) are kept out of this repository.

## Day 1: Mon Oct 26, 2026

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| Power | Chose outlet and surge protector | No heavy loads on the circuit; "protected" light on | Ready |
| SW1 (Catalyst 2960G, WS-C2960G-24TC-L) | First power-on, no network cables | SYST blinked green/amber during self-test; after 5 min SYST green, STAT green, RPS off; fan steady | PASS |
| SRV1 (PowerEdge R730) | Standby check, power button not pressed | Both power supply handles steady green | PASS |
| SRV1 | First full boot | Error UEFI0058: uncorrectable memory error; LCD amber, "error on DIMM_A1, reseat memory" | Fault found |
| SRV1 | Read-only System Setup (F2) inventory | System Event Log: critical multi-bit memory errors on DIMM_A1; 96 GB usable | No settings changed |

## Day 2: Tue Oct 27, 2026

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| SRV1 | Removed cover and cooling shroud; located A1 from Dell's memory diagram | A1: middle bank, 6th socket from CPU 1, white tabs | Done |
| SRV1 | Reseated DIMM_A1 (Samsung 32 GB DDR4-2400 RDIMM) | Contacts clean; ejectors locked | Done |
| SRV1 | Verification boot to System Setup | No memory error; 128 GB usable; front LCD blue | **PASS: fault cleared** |

Note: one verification boot missed the F2 prompt and reached the previous owner's operating system login screen. It was not logged into, no network was connected, and it was shut down cleanly. The drives are wiped in Week 11.

## Day 3: Tue Oct 27, 2026 (evening)

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| Laptop | Set up console access: USB-to-RJ-45 console cable, PuTTY 0.85, serial 9600 baud, 8 data bits, no parity, 1 stop bit, no flow control; session logging to a private folder | Cable detected as a USB serial port | Ready |
| SW1 | Powered on with PuTTY open; watched the full boot over the console | Boot completed; initial configuration dialog appeared (answered `no`) | PASS |
| SW1 | `show version` (read-only) | IOS 15.0(2)SE10a, LAN Base K9; boot loader 12.2(44)SE5; 64 MB RAM. Sanitized copy: [sw1-show-version.md](sw1-show-version.md) | PASS |
| SW1 | Read-only health sweep | Fan, temperature and power OK; no saved configuration; password recovery enabled | PASS |
| SW1 | `show vlan brief` (read-only) | VLANs left by a previous owner were still stored in `flash:vlan.dat` | Finding |
| SW1 | Deleted `flash:vlan.dat`, then reloaded without saving | Only VLAN 1 and built-in VLANs 1002–1005 remain | **PASS: switch factory clean** |

Note: the previous owner's VLAN names and VTP domain are recorded in our private log only.

## Day 4: Wed Oct 28, 2026

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| Laptop | `ipconfig` (read-only) | Home network size confirmed (/24) so lab subnets can avoid it; addresses kept private | PASS |
| GitHub | Drafted lab IP plan v1 and checked every subnet for overlap with the home network | No overlaps. [ip-plan.md](../network/ip-plan.md) | PASS |
| SW1 | `show vtp status` (read-only) | VTP in default Server mode, no domain name | PASS |
| SW1 | Set `vtp mode transparent`; created VLANs 10 MGMT, 20 PLATFORM, 30 APPS, 999 UNUSED | `show vlan brief`: all four active, no ports assigned yet | PASS |
| SW1 | Backed up `show running-config` to the private log, then `copy running-config startup-config` (first save) | `[OK]`; `show startup-config` contains VTP transparent and all four VLANs. [sw1-vlans.md](../network/sw1-vlans.md) | **PASS** |
| PI1 (Raspberry Pi 4, 4 GB) | Wrote Raspberry Pi OS Lite (64-bit) with Raspberry Pi Imager; no Wi-Fi configured; direct Ethernet cable to the laptop only | First boot completed but the Pi had no fixed address | FAIL |
| PI1 | Diagnosed from the laptop and then from the card | Notepad had saved our edit as `network-config.txt`, so cloud-init never read it. It only reads `network-config`, and only on first boot | Root cause found |
| PI1 | Re-imaged; edited `network-config` in place and saved with plain Save | First boot applied 10.20.10.10/24 on `eth0` | PASS |
| PI1 | SSH login from the laptop over the direct cable | Logged in; hostname `pi1`; `wlan0` down with no address; Debian GNU/Linux 13.7 (trixie), kernel 6.18.50, aarch64. [pi1.md](../build/pi1.md) | **PASS** |

## SRV1 hardware (as found)

| Component | Value |
| --- | --- |
| Processors | 2 x Intel Xeon E5-2683 v4 @ 2.10 GHz, 16 cores each (32 cores / 64 threads) |
| Memory | 128 GB ECC DDR4-2400 registered |
| BIOS | 2.19.0 |
| Storage | 2 x 1.92 TB SATA SSD (bays 0–1) |
| Network | Broadcom 5720, 4 x 1 GbE onboard; dedicated iDRAC port |
| Power | 2 x 750 W, redundant |
| Virtualization Technology | Enabled |

## Pending

- FW1 (PA-220): not powered yet. Decision: use the 12 V adapter we already have instead of waiting for the genuine Palo Alto adapter, after checking its output and polarity against Palo Alto's hardware reference
- SW1 hardening (planned): hostname, enable secret, SSH, turn off the built-in web server, set the clock
