# PI1 build log: Raspberry Pi 4 (4 GB)

Role: jump box in the MGMT zone (VLAN 10), the controlled entry point for administering the lab
Built: Day 4 (Wed Oct 28, 2026)

## Result

| Item | Value |
| --- | --- |
| OS | Raspberry Pi OS Lite (64-bit): Debian GNU/Linux 13.7 (trixie), kernel 6.18.50, aarch64 |
| Hostname | `pi1` |
| Address | 10.20.10.10/24 on `eth0`, static, no gateway |
| Wi-Fi | Not configured; `wlan0` is down with no address |
| Remote access | SSH with keys only (password and root login disabled) |
| Connected to | SW1 port Gi0/2 (access VLAN 10, MGMT) |

The Pi has never been on the home network. With no gateway and no Wi-Fi, it can reach only devices in the MGMT VLAN on SW1: the admin laptop and SW1 itself.
## How it was built

1. **Raspberry Pi Imager 2.x** with these settings: Raspberry Pi 4, Raspberry Pi OS Lite (64-bit), hostname `pi1`, a named admin user (not the default `pi`), Wi-Fi left empty, SSH on with password authentication, Raspberry Pi Connect off.
2. **Static address before first boot.** The image's boot partition (`bootfs`) contains a `network-config` file that cloud-init reads on first boot. We replaced its contents with:

   ```yaml
   network:
     version: 2
     renderer: NetworkManager
     ethernets:
       eth0:
         dhcp4: false
         addresses:
           - 10.20.10.10/24
         optional: true
   ```

3. **Laptop side:** set the USB Ethernet adapter manually to 10.20.10.200, subnet mask 255.255.255.0, with no gateway and no DNS. Internet Connection Sharing stays off, so the laptop doesn't bridge the Pi onto the home network.
4. **Boot and test:** `ping 10.20.10.10`, then `ssh <user>@10.20.10.10`.
5. **Read-only checks on the Pi:** `hostname`, `ip -br addr`, `cat /etc/os-release`. Shut down with `sudo poweroff`.

## What went wrong the first time (and the fix)

**Symptom:** the Pi booted, but `ping 10.20.10.10` got "Destination host unreachable." For a while it answered by name over IPv6 link-local, then went silent.

**Root cause:** Windows Notepad saved the edit as a new file, `network-config.txt`. Cloud-init reads only `network-config`, and the original file was entirely commented out, so the Pi had no network instructions. It fell back to DHCP, found no DHCP server on the direct cable, and its connection kept timing out.

**Why we re-imaged instead of fixing the file:** cloud-init applies `network-config` only on the first boot, which had already happened.

**Fix:** re-imaged the card, opened the existing `network-config` in Notepad, replaced its contents, and saved with plain **Save** (not Save As). Before ejecting, we confirmed that no `.txt` file existed and that `network-config` showed a fresh modified time.

**Troubleshooting path that found it:**
1. `ipconfig` showed the laptop side was correct.
2. The Pi's green activity LED showed it had booted.
3. `ping pi1.local` reached it over IPv6, so the cable and OS were fine.
4. The neighbor tables (`netsh interface ipv6 show neighbors`, `arp -a`) showed no IPv4 address on the Pi.
5. Inspecting `bootfs` with file name extensions turned on revealed the `.txt`.

**Lesson:** in Windows File Explorer, turn on **View → Show → File name extensions** before editing configuration files.

## Other notes

- On first boot, the Pi's activity LED stayed dark until the microSD card was reseated firmly.
- Windows `ssh` couldn't resolve `pi1.local` even when `ping` could. Using the address directly avoids the problem.
## Day 5 correction: the Day 4 image was the desktop version

A read-only check on Day 5 showed the Day 4 card held the full desktop Raspberry Pi OS, not Lite: `systemctl get-default` returned `graphical.target`, and the `lightdm` package (a graphical login screen) was installed with 19 services running. The likely cause is that Imager's top OS choice is the desktop version; Lite is under "Raspberry Pi OS (other)".

**Fix (Day 5):** re-imaged with Raspberry Pi OS Lite (64-bit), a new admin username, and the same `network-config` method.

**Verified:** `multi-user.target`; no `lightdm` package; 15 services running; hostname `pi1`; `eth0` at 10.20.10.10/24; `wlan0` down; Debian 13.7 (trixie).

**Lesson:** check what was installed, not just what was intended. One command (`systemctl get-default`) would have caught it on Day 4.

## SSH hardening (Day 5)

**Goal:** only someone holding the laptop's private key can log in to pi1. Passwords no longer work over SSH, and root can't log in at all.

### Key-based login

1. On the laptop, generated an `ed25519` key pair protected by a passphrase (stored in our password manager). The private key never leaves the laptop.
2. Copied the public key to pi1's `~/.ssh/authorized_keys`, with permissions `700` on `~/.ssh` and `600` on `authorized_keys`. SSH refuses keys stored with looser permissions.
3. Confirmed key login worked **before** turning off passwords, so we couldn't lock ourselves out.

### Turning off password login

Created a drop-in file, `/etc/ssh/sshd_config.d/10-tekexec.conf`:

    PasswordAuthentication no
    KbdInteractiveAuthentication no
    PermitRootLogin no

**Why the file name matters:** cloud-init had already written `50-cloud-init.conf`, which sets `PasswordAuthentication yes`. sshd reads the files in this folder in name order and keeps the **first** value it finds for each setting, so `10-` wins over `50-`.

### Verification

- `sudo sshd -T` (prints the settings sshd is actually using) shows `passwordauthentication no`, `kbdinteractiveauthentication no` and `permitrootlogin no`.
- A deliberate password-only login attempt from the laptop was refused with `Permission denied (publickey)`. Here, that error is the success message.

### Account hygiene

- The admin account is a new named user (not `pi`, and not our public GitHub name).
- `sudo` still asks for that user's password, and it's the only account in the `sudo` group.

### Jump box to SW1

pi1 is now the entry point for managing the switch: laptop → pi1 → SW1.

SW1's 2016-era IOS only offers old SSH algorithms that modern OpenSSH turns off by default. Rather than weakening SSH everywhere, pi1's `~/.ssh/config` re-enables them **for SW1 only**:

    Host sw1
        HostName 10.20.10.2
        KexAlgorithms +diffie-hellman-group14-sha1
        HostKeyAlgorithms +ssh-rsa
        Ciphers +aes256-cbc

From pi1, `ssh sw1` reaches the switch. Every other SSH connection keeps the modern defaults.

## Next

Trim unneeded services (avahi-daemon, bluetooth, wpa_supplicant, udisks2 are candidates), check that the system journal persists across reboots, and give pi1 a reliable time source once the firewall provides one.
