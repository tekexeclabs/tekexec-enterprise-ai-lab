# Power-on log

Rules: one device at a time, no settings changed, nothing connected to the home network.
Power: GE surge protector (Type 3 SPD, VPR 400 V, 15 A), plugged directly into a wall outlet.
Private details (Service Tag, serial numbers, MAC addresses) are kept out of this repository.

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

- FW1 (PA-220): not powered; waiting on the genuine Palo Alto adapter (PAN-PA-220-PWR-AC)
