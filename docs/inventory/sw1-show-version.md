SW1 — show version (sanitized)

Device: SW1, Cisco Catalyst 2960G (lab switch) Captured: Oct 27, 2026, over the console port, from user EXEC mode (Switch>), read-only Sanitized: serial numbers, board IDs and MAC address replaced with [REDACTED]; Cisco's export-law boilerplate trimmed

Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE10a, RELEASE SOFTWARE (fc3)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2016 by Cisco Systems, Inc.
Compiled Thu 03-Nov-16 14:21 by prod_rel_team

ROM: Bootstrap program is C2960 boot loader
BOOTLDR: C2960 Boot Loader (C2960-HBOOT-M) Version 12.2(44)SE5, RELEASE SOFTWARE (fc1)

Switch uptime is 9 minutes
System returned to ROM by power-on
System image file is "flash:c2960-lanbasek9-mz.150-2.SE10a.bin"

[Cisco cryptography export notice trimmed]

cisco WS-C2960G-24TC-L (PowerPC405) processor (revision E0) with 65536K bytes of memory.
Processor board ID [REDACTED]
Last reset from power-on
1 Virtual Ethernet interface
24 Gigabit Ethernet interfaces
The password-recovery mechanism is enabled.

64K bytes of flash-simulated non-volatile configuration memory.
Base ethernet MAC Address       : [REDACTED]
Motherboard assembly number     : 73-10015-06
Power supply part number        : 341-0098-02
Motherboard serial number       : [REDACTED]
Power supply serial number      : [REDACTED]
Model revision number           : E0
Motherboard revision number     : A0
Model number                    : WS-C2960G-24TC-L
System serial number            : [REDACTED]
Top Assembly Part Number        : 800-26673-03
Top Assembly Revision Number    : B0
Version ID                      : V03
Hardware Board Revision Number  : 0x01

Switch Ports Model              SW Version            SW Image
------ ----- -----              ----------            ----------
*    1 24    WS-C2960G-24TC-L   15.0(2)SE10a          C2960-LANBASEK9-M

Configuration register is 0xF
What it tells us
Field	Value	Meaning
Model	WS-C2960G-24TC-L, Version ID V03	Matches the chassis label read on Day 1
Software	IOS 15.0(2)SE10a, LAN Base, K9 (crypto)	K9 means SSH is available for secure management (plan Week 6)
Latest available	15.0(2)SE11 (community-reported final release)	We stay on SE10a: downloads need a Cisco contract, and SW1 never faces the internet
Memory	64 MB DRAM, 64 KB NVRAM	NVRAM holds the saved configuration (currently empty)
Ports	24 Gigabit Ethernet + 1 virtual (Vlan1)	Vlan1 is the switch's own management interface
Password recovery	Enabled	If we ever lock ourselves out, Cisco's recovery procedure still works
Boot loader	12.2(44)SE5	The small program that starts IOS
