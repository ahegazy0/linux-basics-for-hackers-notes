# Linux Basics for Hackers
## Module 0 — Getting Started & Setting Up Your Lab

---

## What this module is about

Before anything else, you need somewhere safe to practice. You don't want to break your actual computer while learning. This module is about setting up that safe space — a virtual machine running Kali Linux — and understanding why Linux is the go-to OS for hackers in the first place.

---

## Why Linux?

Windows hides a lot from you. Most things happen behind the scenes and you're not supposed to care about them. Linux is the opposite — everything is visible, everything is configurable, and nothing is locked away.

Two things make Linux special for hacking:

- **Open source** — the entire source code is public. You can read it, change it, rebuild it. Nothing is a black box.
- **Transparent** — you can see exactly what the system is doing at any moment. No hidden processes you can't inspect.

Think of Windows like a car where the hood is welded shut. You can drive it fine, but you can't touch the engine. Linux is the same car with the hood always open and every tool laid out on the table.

---

## What is Kali Linux?

Linux comes in many versions called **distributions** (or "distros"). Ubuntu, Fedora, Mint — these are all Linux, just packaged differently for different purposes.

Kali Linux is a distro built specifically for security professionals. It comes pre-loaded with hundreds of hacking and penetration testing tools so you don't have to hunt them down and install them one by one. It's built on top of **Debian**, which is one of the oldest and most stable Linux distributions out there.

Kali is the industry standard. It's what professionals use, so it's what we'll use.

---

## The Virtual Machine

A Virtual Machine (VM) is a computer running inside your computer. It takes a slice of your hardware — some RAM, some CPU — and uses it to run a completely separate operating system in its own window.

```
Your real computer (Windows or macOS)
└── VirtualBox
    └── Kali Linux VM  ← this is where you practice
```

Your real computer is called the **host**. The VM is called the **guest**. Whatever happens inside the guest stays inside the guest. If you break something, you reset the VM and it's gone. Your host machine never even notices.

This is why professionals always test exploits inside VMs first. If you accidentally run something destructive, the VM takes the hit, not your real system.

---

## VirtualBox and the ISO file

**VirtualBox** is the free software you use to create and run VMs. It's made by Oracle and works on Windows, macOS, and Linux.

**An ISO file** is a disk image — a complete copy of an operating system packaged into one file. When you download Kali Linux, what you're downloading is an ISO. VirtualBox uses that ISO to install Kali inside your VM, the same way you'd install an OS from a physical disc.

---

## Before you install — things to check

**Virtualization must be enabled in BIOS.** This is the most common reason VirtualBox fails to start a VM. Your CPU supports virtualization but it's sometimes turned off by default in the BIOS settings.

- Intel CPUs call it: **Intel VT-x**
- AMD CPUs call it: **AMD-V**

To check: restart your computer and press `DEL`, `F2`, or `F12` (depends on your motherboard) to get into BIOS. Look for a virtualization option and make sure it's enabled.

**RAM allocation.** Don't give the VM too much RAM or your host machine will slow down badly. A simple rule — give 1 GB to the VM for every 4 GB your computer has.

| Your total RAM | Give the VM |
|---|---|
| 4 GB | 1 GB |
| 8 GB | 2 GB |
| 16 GB | 4 GB |

**The root password.** During installation, Kali will ask you to set a root password. Root is the all-powerful admin account — full control over everything in the system. Set something you'll remember, even if it's simple for now. You can change it later.

---

## Setting it up

1. Download VirtualBox from virtualbox.org and install it.
2. Download the Kali Linux 64-bit Installer ISO from kali.org/get-kali.
3. Open VirtualBox → click New → name it "Kali" → type: Linux → version: Debian 64-bit.
4. Assign RAM based on the table above.
5. Create a virtual hard disk, 20 GB minimum.
6. Go to Settings → Storage → click the empty disc icon → attach your ISO file.
7. Start the VM and follow the Kali installer through.
8. Set your username and root password when asked.

You'll know it worked when you hit the Kali login screen with the dragon logo.

---

## Key terms

| Term | What it means |
|---|---|
| Open Source | The code is public — anyone can read or modify it |
| Distribution (distro) | A version of Linux packaged for a specific purpose |
| Debian | The stable Linux base that Kali is built on |
| Virtual Machine (VM) | A fake computer running inside your real one |
| VirtualBox | The software that creates and runs VMs |
| ISO file | A downloadable disk image used to install an OS |
| Host | Your real, physical computer |
| Guest | The VM running inside the host |
| Root | The all-powerful admin user in Linux |

---

## Practice

- Download VirtualBox and the Kali 64-bit ISO
- Create a VM named "Kali" and get it running
- Get to the login screen with the dragon wallpaper — that's the finish line for this module

---

*Up next: Module 1 — The Basics of the Terminal*
