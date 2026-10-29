# SW1 VLAN configuration notes

Device: SW1, Cisco Catalyst 2960G (WS-C2960G-24TC-L), IOS 15.0(2)SE10a
Configured: Day 4 (Wed Oct 28, 2026), over the console port
Source plan: [IP plan v1](ip-plan.md)

## What we changed

| Change | Why |
| --- | --- |
| VTP mode set to **transparent** | SW1 keeps its own VLAN list. It will not learn VLANs from other switches or push its own to them. VTP is how this switch held on to a previous owner's VLANs before Day 3. |
| VLANs 10, 20, 30 and 999 created | The zones from IP plan v1 |
| Configuration saved for the first time | The setup survives a reboot. In transparent mode, the VLANs are also written into the saved configuration. |

## Commands used

```
enable
terminal length 0
show vtp status
configure terminal
vtp mode transparent
vlan 10
 name MGMT
vlan 20
 name PLATFORM
vlan 30
 name APPS
vlan 999
 name UNUSED
end
show vlan brief
show vtp status
show running-config
copy running-config startup-config
show startup-config
```

## Verification

`show vlan brief`:

```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/1, Gi0/2, Gi0/3, Gi0/4
                                                Gi0/5, Gi0/6, Gi0/7, Gi0/8
                                                Gi0/9, Gi0/10, Gi0/11, Gi0/12
                                                Gi0/13, Gi0/14, Gi0/15, Gi0/16
                                                Gi0/17, Gi0/18, Gi0/19, Gi0/20
                                                Gi0/21, Gi0/22, Gi0/23, Gi0/24
10   MGMT                             active
20   PLATFORM                         active
30   APPS                             active
999  UNUSED                           active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

`show vtp status` (excerpt):

```
VTP Operating Mode                : Transparent
Maximum VLANs supported locally   : 255
Number of existing VLANs          : 9
Configuration Revision            : 0
```

Save: `copy running-config startup-config` returned `[OK]`. `show startup-config` contains `vtp mode transparent` and all four VLANs with their names.

VLANs 1002–1005 are built in for legacy network types (FDDI, Token Ring) that this switch doesn't support ("act/unsup"), and they can't be deleted.

## Current state and known gaps

- All 24 ports are still in VLAN 1. Assigning ports to VLANs comes in a later session.
- `interface Vlan1` has no IP address and is shut down, so the switch can't be reached over the network yet.
- Not hardened yet (planned): default hostname, no enable secret, built-in web server on, and no clock source (the timestamps read 1993).
- A full backup of the running configuration is kept in our private log, not in this repository.

## Recovery

To return SW1 to a blank state: `write erase`, then `delete flash:vlan.dat`, then `reload`, answering **no** when asked to save.
