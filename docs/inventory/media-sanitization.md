# Media sanitization: SRV1's previous-owner drives

Standard: NIST SP 800-88 Rev. 2 (September 2025), with techniques from IEEE 2883-2022.

Serial numbers, WWNs and the previous owner's configuration are kept in private records, not here.

## Decision process

1. **Checkpoint (Day 16):** both builders reviewed every drive and signed off that none held our data. All three held only a previous owner's data, so all three were treated as sensitive.
2. **Target level:** Purge where the drive supports it. Destroy (certified service, certificate of destruction) wherever Purge isn't possible. No do-it-yourself destruction.

## Results

| Media | Location | What we could do | Method | Verified by | Final step |
| --- | --- | --- | --- | --- | --- |
| 1 TB SAS HDD | Bay 0 | **Clear** (no self-encryption, so no cryptographic erase) | Old RAID configuration deleted; storage controller one-pass erase | Controller-reported completion | Certified destruction |
| 1 TB SAS HDD (failing) | Bay 2 | Nothing logical: the controller could no longer see the drive | None possible | n/a | Certified destruction |
| 100 GB SATA SSD | Slim optical bay, motherboard SATA | **Clear** (Sanitize not supported by its firmware; Security Erase blocked by the server's start-up "freeze") | Full zero overwrite from a Linux rescue disk booted via iDRAC virtual media | Full read-back: every byte zero, to the last byte | Certified destruction |

## Why Clear plus Destroy

A normal overwrite can't reach an SSD's hidden spare flash, and a hard drive without self-encryption can't do a cryptographic erase. Clear makes the data unreadable by normal means while the drives wait for destruction. Certified destruction provides the final assurance and the paperwork.

## Status

- [x] Checkpoint signed (Day 16)
- [x] All three drives cleared or confirmed unreadable by the controller (Days 16-17)
- [ ] Drives removed from SRV1
- [ ] Certified destruction completed; certificate filed privately
