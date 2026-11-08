# Lab cabling (standard layout)

Verified Day 9 (Tue Nov 3, 2026). Port numbers are the numbers printed beside each jack.

| Cable | SW1 port | Far end | SW1 port mode | Purpose | Verified by |
| --- | --- | --- | --- | --- | --- |
| Yellow Cat6 | Gi0/1 | Admin laptop, USB Ethernet adapter (10.20.10.200) | Access, VLAN 10 | Admin workstation | `show interfaces status`: connected |
| Black Cat6 | Gi0/2 | pi1 (10.20.10.10) | Access, VLAN 10 | Jump box | `show interfaces status`: connected |
| Cat6 | Gi0/3 | FW1 port 2 (ethernet1/2) | Trunk, 802.1q, native 999, allowed 10, 20, 30 | Carries all lab VLANs to the firewall | `show interfaces trunk`; FW1 ethernet1/2 up |
| Cat6 | Gi0/4 | FW1 MGT (10.20.10.3) | Access, VLAN 10 | Firewall management | FW1 `show interface management`: up |
| None | Gi0/5-24 | Unused | Access, VLAN 999, shut down | Locked by default | `show interfaces status`: disabled |

## Firewall side (FW1, PA-220)

| FW1 jack | PAN-OS name | Address | Zone |
| --- | --- | --- | --- |
| Port 2 | ethernet1/2 (Layer 3, no address on the parent) | none | none |
| Port 2, tag 10 | ethernet1/2.10 | 10.20.10.1/24 | mgmt |
| Port 2, tag 20 | ethernet1/2.20 | 10.20.20.1/24 | platform |
| Port 2, tag 30 | ethernet1/2.30 | 10.20.30.1/24 | apps |
| MGT | management | 10.20.10.3/24 | n/a (management plane) |

## Console access

One USB-to-RJ-45 console cable is shared. It connects to SW1's rear CONSOLE port or FW1's front RJ-45 CONSOLE port (the jack below MGT, not the micro-USB one), one device at a time. Both use 9600 baud, 8 data bits, no parity, 1 stop bit, no flow control.

## Nothing connects to the home network

No lab cable reaches the home router. FW1 ports 1 and 3-8 are empty.

## Outside connection (added Day 10)

| Cable | FW1 port | Far end | Settings |
| --- | --- | --- | --- |
| Cat6 | Front jack 1 = ethernet1/1 (verified by link up) | Home router, a LAN port (never its WAN port, never the ISP's box) | Zone untrust; DHCP client with default route; no interface management profile |

The home router has no route to the 10.20.x lab networks, and FW1's outside port answers nothing from the home side (no ping, SSH or HTTPS). Unplugging this one cable disconnects the lab from everything outside it.
