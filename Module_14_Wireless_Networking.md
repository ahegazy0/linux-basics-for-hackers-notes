# Linux Basics for Hackers
## Module 14 — Wireless Networking

---

## What this module is about

Wi-Fi is one of the most common attack surfaces in the real world because it's everywhere and often configured poorly. This module covers how wireless networks actually work, the tools used to inspect and audit them, and how Bluetooth fits into the picture. Everything here should only be used on networks and devices you own or have explicit written permission to test.

---

## How Wi-Fi actually works (the part that matters for this)

When your laptop connects to a router, it's exchanging radio signals on a specific frequency. Normally your Wi-Fi card only pays attention to packets addressed to it — it ignores everything else floating around in the air. This is called **managed mode**.

To do any serious wireless analysis, you need your card to listen to everything — all packets from all devices on all networks within range, not just the ones meant for you. This is called **monitor mode**. It's the equivalent of going from a private conversation to hearing everything in the room.

Not every Wi-Fi adapter supports monitor mode or packet injection. The built-in card on most laptops does not. You need an external USB adapter that specifically supports these features — the Alfa AWUS036ACH and similar Alfa adapters are the standard choice for this kind of work.

---

## The wireless toolkit — aircrack-ng

`aircrack-ng` is a suite of tools, not a single program. The main ones you'll use:

| Tool | What it does |
|---|---|
| `airmon-ng` | Enables and disables monitor mode on your adapter |
| `airodump-ng` | Captures packets and shows all nearby networks |
| `aireplay-ng` | Injects packets into a network (for forcing reconnections, etc.) |
| `aircrack-ng` | Attempts to crack captured WPA handshakes |

They work together as a pipeline. You'll rarely use just one.

---

## Putting your card into monitor mode

First, find your wireless interface name:

```bash
iwconfig
```

Output shows your interfaces. It's usually `wlan0` or similar.

Before enabling monitor mode, kill any processes that might interfere:

```bash
airmon-ng check kill
```

This stops NetworkManager and other background tools that try to manage the interface. If you skip this step, monitor mode often breaks or the interface keeps dropping.

Now enable monitor mode:

```bash
airmon-ng start wlan0
```

Your interface will likely be renamed to `wlan0mon` to indicate it's now in monitor mode. Confirm with `iwconfig` again.

To go back to normal managed mode:

```bash
airmon-ng stop wlan0mon
```

---

## Scanning nearby networks with airodump-ng

With your card in monitor mode, you can capture everything in the air:

```bash
airodump-ng wlan0mon
```

Output shows every network within range:

```
BSSID              PWR  Beacons  #Data  CH   MB   ENC   ESSID
AA:BB:CC:DD:EE:FF  -45      120     34   6  130   WPA2  HomeNetwork
11:22:33:44:55:66  -72       80     12  11   54   WPA2  CoffeeShop_WiFi
```

The columns to understand:

| Column | What it means |
|---|---|
| BSSID | The MAC address of the router |
| PWR | Signal strength — more negative = weaker signal |
| CH | Channel the network is broadcasting on |
| ENC | Encryption type (WPA2, WPA3, WEP, OPN) |
| ESSID | The network name (SSID) you see when connecting |

To focus on a specific network and also see the connected clients:

```bash
airodump-ng --bssid AA:BB:CC:DD:EE:FF --channel 6 -w capture wlan0mon
```

- `--bssid` filters to one specific router
- `--channel` locks to that channel so you don't miss packets while hopping
- `-w capture` writes everything captured to files starting with "capture"

---

## Wi-Fi terms to know

**SSID** — the network name. What you see in the list when you connect.

**BSSID** — the router's MAC address. Unlike the SSID (which the owner can set to anything), the BSSID is tied to the hardware.

**Channel** — Wi-Fi broadcasts on specific channels within the 2.4GHz or 5GHz band. The 2.4GHz band has channels 1–14 (1, 6, and 11 are the standard non-overlapping ones). The 5GHz band has many more.

**WPA2** — the current standard encryption for Wi-Fi networks. Uses AES encryption. Replaced WEP (which was completely broken) and WPA. WPA3 is the newer standard starting to appear on modern routers.

**WEP** — old and completely broken. A network using WEP can be cracked in minutes. If you ever see one in the wild it's a legacy device that hasn't been updated in a very long time.

**Handshake** — when a device connects to a WPA2 network, the router and device exchange a 4-way handshake to verify the password. Capturing this handshake is what WPA2 cracking is based on — you capture it, then try to crack it offline.

---

## Scanning with iwlist

If you just want to see nearby networks without going into monitor mode:

```bash
iwlist wlan0 scan
```

This gives you a basic scan of visible networks — SSID, BSSID, channel, signal strength, encryption type. Less detailed than airodump-ng but doesn't require monitor mode and works fine for reconnaissance without touching anything.

---

## Bluetooth with BlueZ

Bluetooth is a shorter-range wireless protocol used by phones, headphones, keyboards, speakers, smartwatches, and dozens of other devices. The tools for working with Bluetooth on Linux come from the **BlueZ** package.

**Scanning for nearby Bluetooth devices:**

```bash
hcitool scan
```

Output:
```
Scanning...
    AA:BB:CC:DD:EE:FF    John's iPhone
    11:22:33:44:55:66    Sony WH-1000XM4
```

This shows any device in discoverable mode. A device in discoverable mode is actively advertising its presence — this is how you pair new devices. Many people leave their devices permanently discoverable without realizing it.

**Checking if a device is reachable:**

```bash
l2ping AA:BB:CC:DD:EE:FF
```

This pings a Bluetooth device the same way `ping` works for IP addresses — confirms the device is within range and responding.

**Getting device information:**

```bash
hcitool info AA:BB:CC:DD:EE:FF
```

Returns the device name, manufacturer, supported features, and other metadata.

**Scanning with more detail:**

```bash
hcitool lescan
```

`lescan` is for **Bluetooth Low Energy (BLE)** devices — fitness trackers, smart home sensors, IoT devices. These often broadcast continuously and are a growing attack surface.

---

## What you're looking for in wireless recon

When you're doing wireless reconnaissance on a network you're authorized to test, here's what matters:

- **Open networks (OPN)** — no encryption, everything transmitted is readable
- **WEP networks** — broken encryption, crackable in minutes
- **WPA2 networks with weak passwords** — capturable handshake, crackable offline with wordlists
- **Hidden SSIDs** — networks that don't broadcast their name, visible in airodump-ng output as blank ESSID fields but the BSSID is still there
- **Bluetooth devices in discoverable mode** — especially devices that shouldn't be discoverable (keyboards, corporate laptops)

---

## Command reference

| Command | What it does |
|---|---|
| `iwconfig` | Show wireless interfaces and their current mode |
| `iwlist wlan0 scan` | Scan for nearby Wi-Fi networks |
| `airmon-ng check kill` | Kill processes that interfere with monitor mode |
| `airmon-ng start wlan0` | Enable monitor mode |
| `airmon-ng stop wlan0mon` | Disable monitor mode |
| `airodump-ng wlan0mon` | Capture packets and show all nearby networks |
| `airodump-ng --bssid [MAC] --channel [CH] -w out wlan0mon` | Capture from a specific network |
| `hcitool scan` | Scan for discoverable Bluetooth devices |
| `hcitool lescan` | Scan for Bluetooth Low Energy devices |
| `l2ping [MAC]` | Ping a Bluetooth device |
| `hcitool info [MAC]` | Get details about a Bluetooth device |

---

## Practice

- Run `iwlist wlan0 scan` and look at the output — find your own home network, note the channel and encryption type
- Run `hcitool scan` in your room and see what Bluetooth devices show up — check how many are in discoverable mode
- If you have a compatible adapter: put it in monitor mode with `airmon-ng`, run `airodump-ng` for 30 seconds, find your home router in the output and note its BSSID and channel, then stop capture and take the card out of monitor mode

Only do the airodump-ng practice on your own home network. Capturing traffic from networks you don't own is illegal in most countries regardless of whether you do anything with the data.

---

*Up next: Module 15 — Managing the Linux Kernel & Loadable Kernel Modules*
