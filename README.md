# TEK EXEC | Enterprise AI Lab

A build log: Turning retired enterprise hardware into a working hybrid-cloud AI platform, documented step by step.

> **What this is:** A personal learning lab built on end-of-life equipment with synthetic data. It is not a production environment, and nothing here claims production experience.

## Current Status

**Day 5: The jump box is live.**

- Day 1: every device identified from its labels and checked against manufacturer support dates.
- Day 2: fixed a server memory fault by reseating one DIMM.
- Day 3: reached the switch over its console and cleared a previous owner's leftover VLANs.
- Day 4: wrote the lab's IP plan, created its VLANs on the switch, and brought up a Raspberry Pi on a fixed lab address, isolated from the home network.
- Day 5: rebuilt the Pi as a lean server (it had been imaged with the desktop version), switched it to SSH-key-only logins, hardened the switch, and connected the laptop and Pi through it. Admin access now runs laptop → Pi → switch, all inside the management VLAN.

Full record: [power-on log](docs/inventory/power-on-log.md).

## The lab

| Role | Device | Status |
| --- | --- | --- |
| Firewall and router between lab zones | Palo Alto PA-220 | Inventoried |
| Lab switch | Cisco Catalyst 2960G-24TC-L | Inventoried |
| Virtualization server | Dell PowerEdge R730 (Proxmox VE planned) | Inventoried |
| Monitoring / jump box | Raspberry Pi 4 | Inventoried |
| Cloud | AWS | Planned |

Shelved as spares: a second 2960G, a Catalyst 3560G and a Cisco ISR 4331. The lab runs only what earns its place.

## Roadmap

1. Infrastructure: segmented network behind the firewall, Proxmox, backups
2. AI-powered IT Operations Command Center
3. Enterprise Knowledge Hub (RAG with citations)
4. AI business document automation
5. Hybrid cloud deployment on AWS
6. AI cloud cost advisor

## Repository Layout

- `docs/inventory/`: hardware inventory, verified specs and the power-on log
- `docs/network/`: IP plan and switch configuration notes
- `docs/build/`: build logs for each device
- `docs/diagrams/`: architecture diagrams
- `docs/decisions/`: one short file per architecture decision
- `projects/`: application code, added as each project starts
