# Linux Basics for Hackers
## Module 13 — Security & Anonymity

---

## What this module is about

Everything you do online leaves a trace. Every website you visit, every connection your machine makes — your real IP address is attached to it. This module is about understanding how that tracking works and how tools like Tor, proxies, VPNs, and proxychains reduce your exposure. This matters both for protecting yourself and for understanding the infrastructure that privacy-conscious people (and attackers) rely on.

---

## Why your IP address matters

Your IP address is assigned to you by your ISP. Every connection you make to any server on the internet carries it. The server you connect to can see it. Your ISP can see everything you connect to. Any network you're on can see your traffic.

This is the problem anonymity tools solve — they put something between you and the destination so the destination sees a different IP, not yours.

---

## Proxies

A proxy is a server that sits between you and your destination. Instead of connecting directly to a website, you connect to the proxy, and the proxy connects to the website on your behalf. The website sees the proxy's IP, not yours.

```
You → Proxy → Website
```

The website logs the proxy's IP address. Your ISP sees you connecting to the proxy but not what you did from there.

Single proxies are the weakest form of anonymity. The proxy server itself knows your real IP and knows where you're going. If someone subpoenas that proxy or it keeps logs, your activity is traceable.

**Free proxies are almost always a bad idea** for anything sensitive. Many are run specifically to capture traffic — honeypots designed to log everything passing through them. If you wouldn't trust a random stranger to carry your mail, don't trust a random free proxy.

---

## Tor — the onion router

Tor routes your traffic through a chain of three servers called **nodes** or **relays**, each one run by volunteers around the world. Each relay only knows the step before it and the step after — no single relay knows both who you are and where you're going.

```
You → Entry Node → Middle Node → Exit Node → Website
```

The encryption works in layers — like an onion. Your traffic is wrapped in three layers of encryption. Each node peels off one layer to find out where to send it next, but can't read the actual content or see the full path.

The website sees the exit node's IP, not yours. The entry node knows your IP but doesn't know the destination. The middle node knows neither.

**How to use it in Kali:**

```bash
apt install tor
service tor start
```

Or download the Tor Browser, which is a pre-configured Firefox that routes all traffic through Tor automatically.

**The tradeoff:** Tor is slow. Three hops through volunteer servers around the world adds significant latency. It's not suitable for high-bandwidth activities or anything time-sensitive. It's also not a magic shield — poor operational security at the application layer (logging into a personal account, for example) defeats it completely.

---

## Proxychains — forcing tools through proxies

Tor and VPNs protect your browser traffic. But if you're running a terminal tool — `nmap`, `curl`, a custom script — that traffic goes out directly unless you route it through something.

`proxychains` is a tool that intercepts network calls from any program and forces them through a configured chain of proxies. You put it in front of any command and the traffic gets redirected.

```bash
proxychains nmap -sT 192.168.1.1
```

Now nmap's traffic routes through your proxy chain before reaching the target.

**The config file:**

```bash
nano /etc/proxychains.conf
```

At the bottom you'll find the proxy list. Add proxies in this format:

```
socks5  127.0.0.1  9050    ← this is Tor's local port
socks4  10.0.0.1   1080
http    203.0.113.5  3128
```

If you add `127.0.0.1 9050` (Tor's default local port) and have Tor running, proxychains will route everything through Tor. Combined, this means even your terminal tools go through the Tor network.

**Two modes worth knowing in the config:**

- `strict_chain` — traffic must go through every proxy in order; if one is down, the connection fails
- `dynamic_chain` — skips dead proxies automatically and uses whatever is available

For most use cases `dynamic_chain` is more practical.

---

## VPNs

A VPN (Virtual Private Network) creates an encrypted tunnel between you and a VPN server. All your traffic travels through that tunnel, encrypted, before going out to the internet. Your ISP sees only that you're connected to a VPN — not what you're doing.

```
You → [encrypted tunnel] → VPN Server → Internet
```

Compared to Tor:
- VPNs are faster
- Only one server involved — the VPN provider can see your traffic and logs if they choose to
- The VPN provider knows your real IP

Reputable paid VPN providers with audited no-log policies are the standard choice. The key phrase is "no-log policy" — you want a provider that demonstrably does not keep records of what you do.

**VPN + Tor combined** is the approach some people use for maximum layering:

```
You → VPN → Tor Network → Website
```

Your ISP sees VPN traffic. The VPN provider sees you connecting to Tor but not the destination. The Tor exit node has no idea who you are. The website sees the Tor exit node.

The tradeoff is even more latency than Tor alone.

---

## Encrypted communications

Anonymizing your network traffic is one side of it. What you communicate also matters.

**ProtonMail** is an email provider where emails are encrypted end-to-end. Messages between ProtonMail users are encrypted on the server in a way that ProtonMail itself cannot read them. Even under a court order, there's nothing to hand over because the keys only exist on the user's device.

For general messaging, **Signal** operates on the same principle — end-to-end encryption by default, minimal metadata retained.

These are standard tools for journalists, activists, lawyers, and anyone handling sensitive communications. Using encrypted communication is not inherently suspicious — it's basic operational hygiene.

---

## What actually breaks anonymity

Tools are only as good as how you use them. Most anonymity failures are at the human layer, not the technical layer:

- Logging into a personal account (Google, social media) while using Tor — the account itself identifies you regardless of IP
- Reusing usernames across anonymous and non-anonymous activities
- Browser fingerprinting — your browser leaks information about your OS, screen resolution, installed fonts, timezone, etc. even without cookies
- Metadata in files — documents and photos often contain embedded data about the device that created them
- Trusting free or unknown proxies with sensitive traffic

Anonymity is a practice, not just a tool you install.

---

## Command reference

| Command | What it does |
|---|---|
| `service tor start` | Start the Tor service |
| `proxychains [command]` | Run any command through your proxy chain |
| `nano /etc/proxychains.conf` | Edit the proxychains configuration |
| `curl ifconfig.me` | Check what IP address the internet sees |
| `proxychains curl ifconfig.me` | Check your apparent IP through proxychains |

---

## Practice

- Install and start Tor, then run `proxychains curl ifconfig.me` — the IP it returns should not be your real one
- Open `/etc/proxychains.conf` and look at the configuration — add the Tor local port (`socks5 127.0.0.1 9050`) to the proxy list
- Visit a "what is my IP" site directly, then through Tor Browser — compare the two IPs
- Look into ProtonMail if you don't already have an account — it's free and worth using for anything sensitive

---

*Up next: Module 14 — Wireless Networking*
