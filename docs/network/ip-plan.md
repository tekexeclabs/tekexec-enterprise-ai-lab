# Lab IP plan v1

Day 4 · Status: planned, not yet configured

All lab addresses are private (RFC 1918) inside 10.20.0.0/16 and do not overlap the home network.

| VLAN | Name | Subnet | Gateway | Purpose |
|---|---|---|---|---|
| 10 | MGMT | 10.20.10.0/24 | 10.20.10.1 | Admin interfaces: switch, iDRAC, firewall MGT, Proxmox UI, jump box |
| 20 | PLATFORM | 10.20.20.0/24 | 10.20.20.1 | Shared services: monitoring, logging, database, AI gateway |
| 30 | APPS | 10.20.30.0/24 | 10.20.30.1 | Business applications |
| 40, 50 | Reserved | 10.20.40.0/24, 10.20.50.0/24 | — | Future zones |
| 999 | UNUSED | — | — | Parking VLAN for unused switch ports |
| — | AWS VPC (future) | 10.120.0.0/16 | — | Reserved for the cloud network |

## Address layout inside each /24
- .1: firewall gateway
- .2–.9: network devices
- .10–.99: servers and VMs (static)
- .100–.199: reserved for DHCP
- .200–.254: spare

## Design decisions
- The third octet matches the VLAN ID, so any address shows its zone.
- The PA-220 is the gateway for every VLAN, so traffic between zones is inspected and logged.
- VLAN 1 (the switch default) carries no lab traffic.
- The cloud range is reserved now so hybrid routing never hits overlapping networks.
- The firewall's outside interface is a single client on the home network; firewall policy blocks the lab from reaching any home device.
