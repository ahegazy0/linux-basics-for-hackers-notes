# Linux Basics for Hackers
## Module 3 - Managing Networks

---

## Overview

Hacking almost always happens over a network. Before you can do anything on a network, you need to understand your own identity on it - your IP address, your MAC address - and how to manage both. This module covers reading your network info, changing it, and understanding the basics of how DNS works.

![Network Routing](assets/network_routing_diagram_1789213191969.jpg)

---

## Two addresses every device has

Every device on a network carries two identifiers:

**IP address** - this is your logical address on the network. It's assigned by the router (or manually by you) and it's how data knows where to go. Think of it like your street address - it tells the network where you are, but it can change.

**MAC address** - this is burned into the physical network card by the manufacturer. It's a hardware-level identifier, 12 hex characters, unique to every network interface in the world. It looks like this: `00:1A:2B:3C:4D:5E`. Unlike an IP address, you can't permanently change it - but you can spoof it temporarily.

Understanding both is important because when you're on a network, these are what identify you. Knowing how to read and change them is a basic skill.

---

## Essential Commands

### ifconfig

This is your main tool for reading and managing network interfaces. Run it with no arguments and it shows you everything about your current network setup.

```bash
ahegazy0@kali:~$ ifconfig
```

The output shows each network interface your machine has. Common ones:

| Interface | What it is |
|---|---|
| `eth0` | Your wired Ethernet connection |
| `wlan0` | Your wireless (Wi-Fi) connection |
| `lo` | Loopback - the system talking to itself, always 127.0.0.1 |

In the output, look for `inet` - that's your current IP address. Look for `ether` - that's your MAC address.

To see a specific interface only:

```bash
ahegazy0@kali:~$ ifconfig eth0
```

To assign a new IP address to an interface:

```bash
ahegazy0@kali:~$ ifconfig eth0 192.168.1.100
```

To also set the subnet mask at the same time:

```bash
ahegazy0@kali:~$ ifconfig eth0 192.168.1.100 netmask 255.255.255.0
```

To bring an interface up or down:

```bash
ahegazy0@kali:~$ ifconfig eth0 up
ahegazy0@kali:~$ ifconfig eth0 down
```

Bringing an interface down and back up is sometimes needed after making changes.

---

### iwconfig

Like `ifconfig` but specifically for wireless interfaces. Shows Wi-Fi specific info like the network name (ESSID), signal strength, and transmission rate.

```bash
ahegazy0@kali:~$ iwconfig
```

Most of the time you're just reading from this one, not writing to it. But it's useful for confirming which wireless network you're connected to and what your signal looks like.

---

### dhclient

When you manually set an IP address with `ifconfig`, you're no longer getting one automatically from the router. If you want to go back to getting an automatic IP from the network, use `dhclient`.

```bash
ahegazy0@kali:~$ dhclient eth0
```

This sends a request to the network's DHCP server saying "give me an IP address" and it assigns you one. If you changed your IP manually and lost internet, this is usually what fixes it.

You can also use this to request a fresh IP if you think your current one is causing issues:

```bash
ahegazy0@kali:~$ dhclient -r eth0     releases your current IP
ahegazy0@kali:~$ dhclient eth0        requests a new one
```

---

### Changing your MAC address

MAC addresses are meant to be permanent but they're easy to spoof at the software level. The change only lasts until you reboot - after that, the real hardware MAC comes back.

First bring the interface down:

```bash
ahegazy0@kali:~$ ifconfig eth0 down
```

Then set a new MAC:

```bash
ahegazy0@kali:~$ ifconfig eth0 hw ether 00:11:22:33:44:55
```

Then bring it back up:

```bash
ahegazy0@kali:~$ ifconfig eth0 up
```

Run `ifconfig eth0` to confirm the change took. The `ether` line should now show your new MAC.

> The MAC address change only exists in memory. It doesn't survive a reboot. If you need it to persist, there are tools like `macchanger` that handle it more cleanly.

---

### dig

`dig` is for querying DNS - the system that translates domain names like `google.com` into IP addresses. It gives you more detail than just pinging a domain.

```bash
ahegazy0@kali:~$ dig google.com
```

The `ANSWER SECTION` in the output shows you the IP addresses that `google.com` resolves to.

To find mail server records specifically:

```bash
ahegazy0@kali:~$ dig google.com mx
```

`mx` stands for Mail Exchange - this tells you which servers handle email for a domain. Useful for recon.

Other record types worth knowing:

| Record type | What it shows |
|---|---|
| `a` | IPv4 address for the domain |
| `aaaa` | IPv6 address |
| `mx` | Mail servers |
| `ns` | Name servers (which DNS servers are authoritative) |
| `txt` | Text records - often contain verification data |

```bash
ahegazy0@kali:~$ dig google.com ns
```

---

### DNS - a quick note on why it matters

DNS (Domain Name System) is the internet's address book. When you type `google.com`, your computer asks a DNS server "what's the IP for google.com?" and gets an address back. Without DNS, you'd have to remember IP addresses for every site.

Hackers care about DNS for a few reasons. First, DNS queries can reveal a lot about a target's infrastructure - mail servers, subdomains, name servers. Second, DNS can be manipulated - if you can redirect DNS responses, you can send people to the wrong server entirely (this is called DNS poisoning or DNS spoofing, covered in later modules).

For now, knowing how to use `dig` to read DNS records is enough.

---

## Command Reference

| Task | Command |
|---|---|
| View all network interfaces | `ifconfig` |
| View one interface | `ifconfig eth0` |
| Set an IP address | `ifconfig eth0 192.168.1.100` |
| Bring interface down/up | `ifconfig eth0 down / up` |
| View wireless info | `iwconfig` |
| Request IP from DHCP | `dhclient eth0` |
| Release current IP | `dhclient -r eth0` |
| Spoof MAC address | `ifconfig eth0 hw ether 00:11:22:33:44:55` |
| DNS lookup | `dig google.com` |
| Find mail servers | `dig google.com mx` |

---

## Practice

- [ ] Run `ifconfig` and find your IP address and MAC address on `eth0` or `wlan0`
- [ ] Run `dig google.com` and read the answer section - note the IP addresses returned
- [ ] Run `dig google.com mx` and see which servers handle Google's email
- [ ] Try changing your IP to `192.168.1.100`, confirm it with `ifconfig`, then run `dhclient eth0` to get a fresh one from DHCP

> 💡 *For deeper practice, I also recommend completing the end-of-chapter exercises in the official **Linux Basics for Hackers** book.*
---

*Up next: Module 4 - Software Management*
