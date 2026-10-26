# TEK EXEC | Enterprise AI Lab

A build log: Turning retired enterprise hardware into a working hybrid-cloud AI platform, documented step by step.

> **What this is:** A personal learning lab built on end-of-life equipment with synthetic data. It is not a production environment, and nothing here claims production experience.

## Current Status

**Day 1: Inventory and Safety.** Every device identified from its labels and checked against manufacturer support dates.
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

- `docs/inventory/`: hardware inventory and verified specs
- `docs/diagrams/`: architecture diagrams
- `docs/decisions/`: one short file per architecture decision
- `projects/`: application code, added as each project starts
