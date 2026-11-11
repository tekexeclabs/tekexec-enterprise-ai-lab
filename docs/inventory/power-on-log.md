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

## Day 5: Thu Oct 29, 2026

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| PI1 | Linux basics (read-only): checked the running system | Desktop packages present: the Day 4 card was the desktop image, not Lite | Finding |
| PI1 | Re-imaged with Raspberry Pi OS Lite (64-bit); same static address method | Debian 13.7 (trixie), multi-user target, no desktop packages | PASS |
| PI1 | Created a laptop ed25519 key (passphrase-protected); installed it in authorized_keys (700/600) | Key login works | PASS |
| PI1 | Drop-in 10-tekexec.conf: password, keyboard-interactive and root login off | `sshd -T` confirms all three; password-only login refused with `Permission denied (publickey)` | **PASS** |
| SW1 | Hardening: hostname, local admin user, enable secret, RSA 2048 keys, SSH v2 only, console and VTY login required, idle timeouts, built-in web servers off, time zone set | Details: [sw1-hardening.md](../network/sw1-hardening.md) | PASS |
| SW1 | Management address 10.20.10.2/24 on Vlan10; Vlan1 shut down | Vlan10 up/up | PASS |
| SW1 | Gi0/1 (laptop) and Gi0/2 (pi1): access VLAN 10, portfast. Gi0/3–24: VLAN 999, shut down | Only the two used ports are active | PASS |
| LAPTOP, PI1 | Cabled into SW1 Gi0/1 and Gi0/2 | Laptop reaches pi1 by SSH through the switch | PASS |
| PI1 | `~/.ssh/config` entry allowing SW1's older SSH algorithms for SW1 only | `ssh sw1` from pi1 reaches the switch | **PASS: jump box working** |
| SW1 | `copy running-config startup-config` | `[OK]` | PASS |

Note: a credential-handling mistake during SW1 hardening exposed its passwords. Risk accepted while SW1 was isolated; both passwords were rotated at the start of Day 7.

## Day 6: Fri Oct 30, 2026

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| FW1 (PA-220) | Adapter check, unpowered: compared a third-party 12 V adapter with Palo Alto's specs and FW1's label | 12 V DC, 3 A (FW1 needs 1.75 A), center-positive, threaded lock fits PWR 1; not Palo Alto's part | HOLD |
| FW1 | First power-on on PWR 1 (before the planned multimeter polarity test) | PWR LED green; no smell, sound or heat | PASS: supervised use only |
| FW1 | Console: RJ-45 CONSOLE, 9600 baud 8N1, no flow control; power-cycled to capture the full boot | Bootloader → PAN-OS; filesystem checks and self-tests passed; login prompt reached | PASS |
| FW1 | Waited for the STAT LED | Yellow (booting) for about 25 minutes, then green | PASS |
| FW1 | First login with factory credentials; forced password change | Logged in | PASS |
| FW1 | Read-only inventory: system info, licenses, environmentals | PAN-OS 9.1.15-h1; no licenses; factory-default management settings; all environmental alarms false; input rail 12.00 V | PASS. Details: [fw1.md](../build/fw1.md) |
| FW1 | `show config diff` (read-only) | New password pending, not yet committed | Finding |
| FW1 | Set a fresh admin password, then `commit` | `show config diff` empty afterwards | **PASS** |
| FW1 | `show running security-policy`, `show interface all` (read-only) | Factory-default rules only; just ethernet1/1–1/2 configured, both down | PASS: no reset needed |
| FW1 | `request shutdown system`, then power removed | "System halted" | PASS |


## Day 7: Sat Oct 31, 2026

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| SW1 | Checked supported hash types (`enable algorithm-type ?`) | scrypt available; Day 5 had assumed type 5 (MD5) was the maximum | Finding |
| SW1 | Rotated enable secret and local user password (exposed Oct 29), hashed as type 9 (scrypt); PuTTY logging paused while typing secrets | Enable and console login tested with the new credentials before saving; `copy running-config startup-config` [OK] | **PASS** |
| GitHub | Audited the repository: file paths, links, and a scan for hashes, usernames, home addresses, serials and fingerprints | No private details found; fixed a filename case mismatch, stale notes and diagram labels | PASS |
| FW1 | Read-only checks: interfaces, clock, virtual wire, zones, system settings | Factory default-vwire bridges ethernet1/1 (untrust) and ethernet1/2 (trust); clock on US/Pacific and about 1 hour slow; Telnet and HTTP management already disabled | Finding |
| FW1 | `show config diff`, then `request shutdown system` | Empty diff; nothing changed; clean halt | PASS |
| Network | Wrote the Week 7 link plan: trunk FW1 ethernet1/2 ↔ SW1 Gi0/3, MGT ↔ Gi0/4 at 10.20.10.3, untrust ethernet1/1 ↔ home router | [fw1-sw1-link-plan.md](../network/fw1-sw1-link-plan.md) | PASS |

## Day 8: Mon Nov 2, 2026

Built both ends of the trunk between SW1 and FW1. SW1 now carries VLANs 10, 20 and 30 on one port, and FW1 has a Layer 3 subinterface and security zone for each VLAN. No cables are connected yet; that's the next session.

### SW1: trunk to FW1 and FW1 management port

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| SW1 | Read-only check: `show running-config interface` for Gi0/3 and Gi0/4, `show running-config \| include default-gateway`, `dir flash:` | Gi0/3 and Gi0/4 in VLAN 999 and shut down; no default gateway set | PASS: clean starting state |
| SW1 | Safety copy: `copy running-config flash:sw1-before-day8.cfg` | Copy saved to flash | Done |
| SW1 | Gi0/3: `switchport mode trunk`, native VLAN 999, allowed VLANs 10,20,30, `switchport nonegotiate`, `no shutdown` | Two typos rejected by the switch with `% Invalid input`; nothing applied until retyped correctly | Done |
| SW1 | Gi0/4: access VLAN 10, `spanning-tree portfast`, `no shutdown` | Standard portfast warning (single-host ports only) | Done |
| SW1 | `ip default-gateway 10.20.10.1` (FW1's address on VLAN 10) | Accepted | Done |
| SW1 | Verified with `show interfaces Gi0/3 switchport`, `show interfaces status`, `show vlan brief` | Trunk administratively on, operationally down (no cable), negotiation off, native 999, allowed 10,20,30; Gi0/4 in VLAN 10 | PASS |
| SW1 | `copy running-config startup-config` | `[OK]` | **PASS: saved** |

Note: while unplugged, Gi0/3 still appears under VLAN 999 in `show vlan brief`. That's its dormant access setting; it should disappear from that list once the trunk is connected and up. Details: [sw1-vlans.md](../network/sw1-vlans.md).

### FW1: VLAN subinterfaces, zones and management address

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| FW1 | Boot and read-only check: `show clock`, `show system info`, `show interface all`, `show zone`, `show network virtual-router` | Clock correct (US/Eastern, EST); management port on its factory default address; no zones, profiles or subinterfaces | PASS |
| FW1 | Configured ethernet1/2 as Layer 3 (no IP on the parent port) with subinterfaces ethernet1/2.10, .20 and .30, VLAN tags 10, 20 and 30, addresses 10.20.10.1/24, 10.20.20.1/24 and 10.20.30.1/24 | Pasting several lines into the 9600-baud console garbled input; .20 received an extra invalid address (10.20.20.1/2) | Mistake |
| FW1 | Zones `mgmt`, `platform`, `apps` (one per subinterface); all three added to virtual router `default`; interface management profile `allow-ping` (ping only) on ethernet1/2.10 only; management port moved to 10.20.10.3/24, gateway 10.20.10.1 | Settings in the candidate configuration | Done |
| FW1 | `validate full`, then `commit` | Validation result read while the job was still running; both commits failed: "address 10.20.20.1/2 ... is invalid" | FAIL: caught before going live |
| FW1 | `set cli pager off`; deleted 10.20.20.1/2; read back interfaces, zones, virtual router, profile and system settings line by line | All correct; .20 has only 10.20.20.1/24 | Fixed |
| FW1 | `validate full`, `commit` | Configuration committed successfully | **PASS** |
| FW1 | `show interface all`, `show system info` | .10/.20/.30 in zones mgmt/platform/apps, virtual router default, correct tags and addresses; ethernet1/2 down (no cable); management port 10.20.10.3 | **PASS: committed** |
| FW1 | `request shutdown system` | System halted; powered off | Done |

### What we learned

- **The commit is a safety net.** PAN-OS checks the whole configuration before making it live, and it refused an invalid address instead of applying a broken setup.
- **One line at a time on a console.** The firewall's console runs at 9600 baud. Pasting a block of commands overran it, dropped characters (`/24` became `/2`), and sent stray keystrokes into the output pager.
- **Read the right job, and wait for it.** A validation result is final only when its status is `FIN`. Check the job ID the command just printed, not an older one.
- **Take the "before" snapshot once.** A second save under the same name overwrote it. The earlier Day 7.5 snapshot served as the rollback point instead.
- **Firewall zones start closed.** With no security rules yet, FW1's built-in defaults allow traffic within a zone and deny traffic between zones. VLANs 10, 20 and 30 are already isolated from each other until rules are written.

## Day 9: Tue Nov 3, 2026

The laptop and pi1 moved onto VLAN 10 through SW1, and SW1 was connected to FW1 by a VLAN trunk. FW1 answered its first ping from the lab. No configuration was changed on either device.

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| SW1, PI1, Laptop | Cable trace; laptop USB Ethernet adapter to SW1 Gi0/1, pi1 to Gi0/2 | Port 2 stayed dark until pi1 was powered on; then ports 1 and 2 green | Done |
| SW1 | `show ip interface brief` (Vlan lines) | Vlan10 10.20.10.2 up/up; Vlan1 administratively down by design | PASS |
| SW1 | `show interfaces status` | Gi0/1 and Gi0/2 connected, VLAN 10, 1000 Mb/s full duplex; Gi0/3 trunk and Gi0/4 not connected yet; Gi0/5-24 disabled in VLAN 999 | PASS |
| Laptop, PI1 | Ping SW1 (10.20.10.2) and pi1 (10.20.10.10) | 4/4 replies each, 0% loss; first reply slower while ARP resolved | PASS |
| PI1, SW1 | SSH from pi1 to SW1; `show privilege`, `show users` | Privilege 15; one SSH session from 10.20.10.10 | **PASS: first network management of SW1** |
| FW1 | Console moved to FW1; `show interface management` and `show interface ethernet1/2` (before) | Both links down (no cables) | PASS (look only) |
| FW1 | `show deviceconfig system service` (read-only, configure mode, exited without changes) | Telnet and HTTP disabled; ping, SSH and HTTPS still allowed on MGT (factory defaults) | Recorded |
| SW1, FW1 | Cable: SW1 Gi0/4 to FW1 MGT | MGT link 1000/full/up; laptop ping to 10.20.10.3: 4/4 | PASS |
| SW1, FW1 | Cable: SW1 Gi0/3 (trunk) to FW1 port 2 | ethernet1/2 link 1000/full/up, confirming the jack labeled 2 is ethernet1/2 | PASS |
| SW1 | `show interfaces trunk` | Gi0/3 mode on, 802.1q, trunking, native VLAN 999, VLANs 10, 20, 30 allowed and forwarding | PASS |
| PI1, FW1 | pi1 ping to 10.20.10.1; FW1 `ping source 10.20.10.1 host 10.20.10.10` | 4/4 and 19/19, 0% loss | **PASS: first traffic through the trunk to FW1** |
| PI1 | SSH to SW1 by address instead of the saved shortcut | "No matching key exchange method": SW1 offers only older SHA-1 methods that current OpenSSH refuses by default | Finding: use the saved `sw1` SSH shortcut |
| FW1 | Reviewed ethernet1/2 counters | 51 receive errors on the parent interface; likely untagged spanning-tree and CDP frames from SW1, dropped because the parent has no untagged subinterface. Ping traffic unaffected | Watch |
| All | Shutdown: FW1 `request shutdown system`, pi1 `sudo poweroff`, SW1 off at the strip | Clean shutdown; all cables left in place | Done |

Notes:
- pi1 has no time source (no internet by design), so its clock is wrong. A lab time source is planned before monitoring starts, because log timestamps matter.
- Standard cabling is now recorded in [lab-cabling.md](../network/lab-cabling.md).

## Day 10: Sat Nov 7, 2026

Goal: finish Week 7 by connecting FW1's outside port to the home router, then start Week 8 with FW1's first security rules, proven with real traffic and the traffic log.

### Session 1: outside interface, built while unplugged

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| FW1 | Read-only checks: application content version, zones, virtual wires, ethernet1/1 | Zones mgmt, platform, apps only; no virtual wire; ethernet1/1 unconfigured | PASS |
| Laptop | Accidental paste into the FW1 console; closed PuTTY and reconnected | Reset the draft with `load config from running-config.xml`; nothing had been committed | Recovered |
| FW1 | Snapshot `day10-before-s1.xml`; ethernet1/1 set to Layer 3 DHCP client with default route; added to virtual router `default`; new zone `untrust`; no interface management profile | `validate full` FIN/OK; commit FIN/OK | PASS |
| FW1 | `show interface ethernet1/1` | Zone untrust, management profile N/A, link down (no cable yet) | PASS |

### Session 2: connect, then test from both sides

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| FW1 | Outside cable: home router LAN port to FW1 port 1 | 1000/full/up; address received by DHCP; default route via the home router | PASS |
| FW1 | Ping from the outside interface to 1.1.1.1 | 4/4 replies, about 5 ms | PASS |
| Laptop (home Wi-Fi) | Ping FW1's outside address; TCP 443 test; ping 10.20.10.1 with the lab cable unplugged | All failed | PASS: home network cannot reach FW1 or the lab |
| PI1 | Default gateway 10.20.10.1 and DNS 1.1.1.1 / 9.9.9.9 set with nmcli; cloud-init network config disabled | `default via 10.20.10.1`; resolv.conf correct | PASS |
| PI1 | Ping 1.1.1.1 and the home router | 0 of 4 replies each | PASS: built-in interzone deny blocks the lab |
| FW1 | `show log traffic` | No entries | Finding: the built-in deny doesn't log, so blocks were invisible |

### Sessions 3 and 4: first security rules, NAT and proof

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| FW1 | Snapshot `day10-before-s3.xml`; address objects `home-lan` (home network, kept private) and `pi1` (10.20.10.10/32) | Created | Done |
| FW1 | Security rules, top to bottom: `deny-lab-to-home`, `allow-pi1-updates` (dns, ntp, ping, web-browsing, ssl, apt-get on their default ports), `deny-lab-outbound-other`; all log at session end | Created | Done |
| FW1 | NAT rule `nat-lab-outbound`: lab zones to untrust, dynamic IP and port on ethernet1/1 | Created | Done |
| FW1 | Commit (job 5). The separate validate job was still running; a commit runs its own full validation | FIN/OK, no warnings | PASS |
| PI1 | Time sync (automatic) | `ntp-base` allowed within seconds of the commit; clock synchronized; time zone set to America/New_York | PASS |
| PI1 | Ping 1.1.1.1; DNS lookup of deb.debian.org; HTTPS to www.debian.org | 4/4; resolved; HTTP/2 200 | PASS |
| PI1 | `sudo apt update` (package list only) | App-ID identified it as `apt-get`; 35 packages upgradable, not installed yet | PASS |
| PI1 | Ping the home router | 0/4; logged under `deny-lab-to-home` | PASS |
| PI1 | SSH to github.com | Timed out; 5 retries logged under `deny-lab-outbound-other` | PASS |
| FW1 | `show log traffic direction equal backward` | Every allow and deny recorded with its rule name | **PASS: Day 10 proven** |

Details: [FW1 outside interface](../network/fw1-outside.md) · [FW1 security policy and NAT](../network/fw1-policy.md) · [Decision record](../decisions/2026-11-07-outside-connection-and-first-rules.md)

Note: home network addresses, FW1's outside address and usernames are kept in private notes.

## Day 11 Sun Nov 9, 2026
- Security-zone diagram v2 in `docs/diagrams/`: add the untrust zone, the one allowed path from pi1, and the two blocked directions
- LinkedIn posts for Days 9 and 10
- Lab DHCP and DNS design (decision record): addresses for VMs on PLATFORM and APPS, name lookups inside the lab, the lab time source, and the IPv6 stance
- PI1: confirm the default gateway survives a reboot; install the 35 pending updates

## Day 12: Mon Nov 9, 2026

| Device | Action | Observed | Result |
| --- | --- | --- | --- |
| SRV1 | Pre-flight placement check (no power) | Desk beside the rack, rated well above the server's weight; front and rear clear; no direct sunlight; 5 ft Cat6 reaches SW1 | PASS |
| SRV1 | Rear panel matched to Dell's R730 Owner's Manual | All 11 back-panel items located; PCIe slots 1-7 empty. [srv1.md](../build/srv1.md) | PASS |
| SRV1 | Internal components identified from photos | Broadcom 5720 4-port 1 GbE network daughter card; Mini-PERC with cache battery (model to confirm); 2.5" 8-bay backplane; 6 fans | PASS |
| SRV1 | Connection and undo plan reviewed | Planned: iDRAC 10.20.10.20 on SW1 Gi0/5 (VLAN 10); Gi0/6 reserved for the NIC 1 trunk. Open: iDRAC license, PERC model | Done |
| ~12:15 PM | FW1 | Software | `check pending-changes` (configure mode; unknown in operational mode) | no | PASS: draft matches live config |
| ~12:20 PM | FW1 | Software | Web UI: Device > Setup > Operations > Load named configuration snapshot (list only, Cancel) | day10-after.xml present | PASS: no new snapshot needed; FW1 unchanged |
| ~12:25 PM | FW1 | Software | `show network interface ethernet ethernet1/1` and `ethernet1/2` (configure mode, read-only) | No IPv6 on any interface; subinterfaces .10/.20/.30 match IP plan; allow-ping only on .10 | PASS |
| ~12:30 PM | FW1 | Software | Web UI: Network > DHCP > DHCP Relay (Add, Cancel) | ethernet1/2.30 offered as a relay interface | PASS: relay available for Week 13 |
| ~12:32 PM | PI1 | Software | `timedatectl status`, `ip -6 addr show eth0`, `apt-cache policy chrony` | Synced, NTP active, no RTC; link-local IPv6 only; chrony not installed, candidate 4.6.1-3+deb13u2 | PASS |
| ~12:37 PM | SW1 | Software | `show clock detail` (via `ssh sw1` from pi1) | *1993 default time, no time source; EST and US summer-time rule configured | Finding: needs pi1 as time source (Week 12) |
| SW1 | Safety copy, then Gi0/5 moved from locked VLAN 999 to VLAN 10 | Port connected at 1 Gbps | PASS |
| SRV1 iDRAC | Cabled to SW1 Gi0/5; ping, HTTPS check, web login | All succeeded from the admin laptop | PASS |
| SRV1 iDRAC | License check in the web UI | iDRAC8 Enterprise | Verified |
