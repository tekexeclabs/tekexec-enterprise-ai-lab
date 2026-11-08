# FW1 outside interface

Device: FW1, Palo Alto PA-220, PAN-OS 9.1.15-h1
Configured: Day 10 (Sat Nov 7, 2026), over the console
Related: [FW1 security policy and NAT](fw1-policy.md) · [Lab cabling](lab-cabling.md)

## Design

| Decision | Why |
| --- | --- |
| ethernet1/1 is the only outside port, in zone `untrust` | One door to the outside, so every lab connection to the internet passes the firewall |
| Address by DHCP from the home router | The home router owns its network's addresses; a fixed address could collide with a family device |
| No interface management profile | Nothing on the home network can ping, SSH to, or open FW1's admin page |
| Configured and committed while unplugged | The whole design was checked before anything outside could reach it |

## Configuration

```
ethernet1/1 {
  layer3 {
    dhcp-client {
      enable yes;
      create-default-route yes;
    }
  }
  comment "OUTSIDE to home router";
}
```

```
untrust {
  network {
    layer3 ethernet1/1;
  }
}
```

ethernet1/1 is also a member of virtual router `default`, alongside the lab subinterfaces ethernet1/2.10, .20 and .30.

## Routing table after connecting (sanitized)

```
destination          nexthop              metric flags  interface
0.0.0.0/0            <HOME-ROUTER>        10     A S    ethernet1/1
10.20.10.0/24        10.20.10.1           0      A C    ethernet1/2.10
10.20.20.0/24        10.20.20.1           0      A C    ethernet1/2.20
10.20.30.0/24        10.20.30.1           0      A C    ethernet1/2.30
<HOME-SUBNET>        <FW1-OUTSIDE>        0      A C    ethernet1/1
```

`A S` on the default route is how PAN-OS lists a route created by the DHCP client's `create-default-route` option.

## Tests (before any security rules existed)

| From | To | Expected | Result |
| --- | --- | --- | --- |
| FW1 outside interface | 1.1.1.1 | Reply | 4/4, about 5 ms |
| Home network (laptop on Wi-Fi) | FW1 outside address, ping | No reply | Timed out |
| Home network | FW1 outside address, TCP 443 | No connection | Failed |
| Home network | 10.20.10.1 | No reply | Timed out |
| pi1 (gateway 10.20.10.1) | 1.1.1.1 and the home router | Blocked by the built-in interzone deny | 0 of 4 each, nothing logged |

The last row is why the next step was logged rules: the built-in deny blocks silently.

## Recovery

Unplug the outside cable for an instant disconnect, or `load config from day10-before-s1.xml`, then validate and commit.
