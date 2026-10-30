# SW1 hardening notes

Device: SW1, Cisco Catalyst 2960G (WS-C2960G-24TC-L), IOS 15.0(2)SE10a
Configured: Day 5 (Thu Oct 29, 2026), over the console port
Builds on: [SW1 VLAN notes](sw1-vlans.md) and [IP plan v1](ip-plan.md)

## What changed

| Change | Why |
| --- | --- |
| Hostname `SW1` | Matches the lab's device tags and diagrams |
| `no ip domain-lookup` | A mistyped command no longer freezes the console while the switch looks it up as a hostname |
| `ip domain-name lab.home.arpa` | Required before SSH keys can be generated. `home.arpa` is reserved for home networks |
| Enable secret and a local admin account | Privileged mode and every login now need credentials (kept in our password manager) |
| RSA 2048 keys, `ip ssh version 2` | Encrypted remote management |
| VTY lines 0–15: `login local`, `transport input ssh`, 10-minute idle timeout | Remote access by SSH only, with a named account |
| Console: `login local`, 15-minute idle timeout | Physical access also needs the account |
| `no ip http server`, `no ip http secure-server` | Turns off the built-in web page |
| `interface Vlan10` 10.20.10.2/24 | Management address in the MGMT VLAN |
| Time zone EST with EDT summer time | Readable log timestamps |

## Port plan

| Port | Device | VLAN | State |
| --- | --- | --- | --- |
| Gi0/1 | Admin laptop | 10 MGMT | Access, portfast, on |
| Gi0/2 | pi1 (jump box) | 10 MGMT | Access, portfast, on |
| Gi0/3–24 | Unused | 999 UNUSED | Shut down |

## Commands (credentials replaced with placeholders)

```
configure terminal
hostname SW1
no ip domain-lookup
ip domain-name lab.home.arpa
enable secret <enable-secret>
username <admin-user> privilege 15 secret <admin-password>
crypto key generate rsa modulus 2048
ip ssh version 2
line vty 0 15
 login local
 transport input ssh
 exec-timeout 10 0
line con 0
 login local
 exec-timeout 15 0
exit
no ip http server
no ip http secure-server
interface vlan 10
 ip address 10.20.10.2 255.255.255.0
 no shutdown
interface range GigabitEthernet0/1 - 2
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown
interface range GigabitEthernet0/3 - 24
 switchport mode access
 switchport access vlan 999
 shutdown
end
copy running-config startup-config
```

## Verification

- `show ip ssh`: SSH enabled, version 2.0
- `show ip interface brief`: Vlan10 10.20.10.2 up/up once a VLAN 10 port is connected
- Console login and `enable` both tested **before** saving
- SSH from pi1 reached `SW1#`; `show users` listed the console session and a VTY session from 10.20.10.10

## Legacy SSH (scoped to this switch)

This IOS release only offers SHA-1 key exchange, `ssh-rsa` host keys and CBC ciphers, which modern OpenSSH disables by default. We allowed them **for SW1 only** in pi1's `~/.ssh/config`:

```
Host sw1
    HostName 10.20.10.2
    User <admin-user>
    KexAlgorithms +diffie-hellman-group14-sha1
    HostKeyAlgorithms +ssh-rsa
    Ciphers +aes256-cbc
```

Every other SSH connection keeps modern defaults. This is acceptable only because SW1's management address sits on an isolated management VLAN.

## Known gaps

- Secrets are stored as type 5 (MD5) hashes, the strongest this release supports. Credentials will be rotated before SW1 connects to the firewall.
- No battery-backed clock or time source yet: the clock resets at every reboot (set by hand until the firewall provides time).
- No default gateway yet; it will be 10.20.10.1 on the PA-220.
- A leftover self-signed certificate entry from the old web server remains in the config (harmless; to be removed).

## Recovery

Before saving: power-cycle to return to the last saved configuration. After saving: password recovery is enabled on this switch.
