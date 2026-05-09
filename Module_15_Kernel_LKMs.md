# Linux Basics for Hackers
## Module 15 — The Kernel & Loadable Kernel Modules

---

## What this module is about

The kernel is the core of the operating system. Everything else — your terminal, your programs, your filesystem — sits on top of it. This module is about understanding what the kernel does, how to inspect and adjust its settings, and how modules work as a way to extend kernel functionality without rebooting.

---

## What the kernel actually does

The kernel sits between your software and your hardware. When a program wants to read a file, send data over the network, or draw something on the screen, it doesn't talk to the hardware directly — it asks the kernel, and the kernel handles it.

This makes the kernel the most privileged piece of software on the entire system. It runs in what's called **kernel space**, which is completely separate from the **user space** where your programs run. A process in user space can only do what the kernel allows. The kernel itself has no such restrictions — it can access any memory, any hardware, anything.

This is also why it's such a target. Code running in the kernel runs with absolute authority over the machine.

---

## Checking your kernel version

```bash
uname -a
```

Output:
```
Linux kali 6.1.0-kali9-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.27-1kali1 x86_64 GNU/Linux
```

Reading this from left to right:
- `Linux` — the OS
- `kali` — the hostname
- `6.1.0-kali9-amd64` — the kernel version and architecture
- `x86_64` — confirms this is a 64-bit kernel

The architecture part (`x86_64` vs `i686`) is how you tell 64-bit from 32-bit.

---

## Loadable Kernel Modules (LKMs)

The kernel can't be recompiled and replaced every time you need to support a new piece of hardware. Instead, it supports **loadable kernel modules** — chunks of kernel code that can be inserted and removed while the system is running, without a reboot.

Drivers are the most common example. When you plug in a USB Wi-Fi adapter, the kernel loads the appropriate driver module to communicate with it. When you unplug it, the module can be removed. The kernel itself doesn't change — the module just extends it temporarily.

Because modules run in kernel space, they have the same absolute authority as the kernel itself. This is why they're also used to hide rootkits — malicious code inserted as a module can intercept system calls, hide processes, hide files, and do it all from a level the OS itself can't easily inspect.

---

## Listing loaded modules

```bash
lsmod
```

Shows every module currently loaded into the kernel:

```
Module                  Size  Used by
bluetooth             663552  2 btusb
snd_hda_intel         57344  3
usbcore              286720  5 btusb,xhci_hcd
```

The columns are:
- `Module` — the module name
- `Size` — how much memory it's using
- `Used by` — other modules or processes depending on it

You'll typically see dozens of modules loaded. Most are hardware drivers — audio, USB, display, network.

---

## Getting information about a module

```bash
modinfo bluetooth
```

Output includes:
- `filename` — where the module file lives on disk
- `description` — what it does
- `author` — who wrote it
- `depends` — other modules it requires to function
- `vermagic` — which kernel version it was compiled for

The `depends` field is particularly useful. Modules often have dependencies — other modules that must be loaded first. `modinfo` shows you that chain.

---

## Loading and removing modules

**Add a module:**

```bash
modprobe bluetooth
```

`modprobe` is the right tool for this. Unlike `insmod` (the lower-level alternative), modprobe automatically resolves and loads any dependencies the module needs. If `bluetooth` requires two other modules to be loaded first, modprobe handles that automatically.

**Remove a module:**

```bash
modprobe -r bluetooth
```

The `-r` flag removes it. Again, modprobe handles dependencies — it won't remove a module that something else still depends on.

---

## sysctl — adjusting kernel settings at runtime

The kernel exposes a large number of configurable settings through a virtual filesystem at `/proc/sys`. The `sysctl` command lets you read and change these settings while the system is running.

**View all current kernel settings:**

```bash
sysctl -a
```

That's a long list. To look at a specific one:

```bash
sysctl net.ipv4.ip_forward
```

Output:
```
net.ipv4.ip_forward = 0
```

This particular setting — IP forwarding — controls whether the system will forward network packets from one interface to another. By default it's off. When it's off, your machine receives a packet, decides it's not addressed to itself, and drops it.

**Turn on IP forwarding:**

```bash
sysctl -w net.ipv4.ip_forward=1
```

The `-w` flag means "write." Now the kernel will forward packets between interfaces. This is required for Man-in-the-Middle setups — your machine needs to be in the middle of traffic flow, receiving packets meant for someone else and forwarding them onward.

**Making a sysctl change permanent:**

Like environment variables, `sysctl -w` only lasts until the next reboot. To make it permanent, add it to `/etc/sysctl.conf`:

```
net.ipv4.ip_forward = 1
```

Then apply without rebooting:

```bash
sysctl -p
```

**Be careful with sysctl.** Some settings control fundamental kernel behavior. Changing the wrong value can cause network problems, instability, or worse. Always know what a setting does before changing it.

---

## The /proc filesystem

Worth knowing alongside sysctl — `/proc` is a virtual filesystem that exists only in memory. It's the kernel exposing information about itself and running processes in file form.

```bash
ls /proc
```

You'll see numbered directories — one for each running process (the number is the PID). You'll also see files like `cpuinfo`, `meminfo`, and `version` that expose hardware and kernel data.

```bash
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/version
```

These are not real files on disk. They're generated on the fly by the kernel every time you read them. This is the "everything is a file" philosophy taken to its logical conclusion — even dynamic kernel data is presented as files.

---

## Command reference

| Command | What it does |
|---|---|
| `uname -a` | Show full kernel version and architecture |
| `lsmod` | List all currently loaded kernel modules |
| `modinfo [name]` | Show details about a module |
| `modprobe [name]` | Load a module (and its dependencies) |
| `modprobe -r [name]` | Remove a module |
| `sysctl -a` | Show all kernel parameters |
| `sysctl [parameter]` | Show a specific kernel parameter |
| `sysctl -w [param=value]` | Change a kernel parameter at runtime |
| `sysctl -p` | Apply changes from /etc/sysctl.conf |
| `cat /proc/cpuinfo` | CPU information from the kernel |
| `cat /proc/meminfo` | Memory information from the kernel |

---

## Practice

- Run `uname -a` and identify the kernel version and whether it's 32-bit or 64-bit
- Run `lsmod` and scroll through — count roughly how many modules are loaded
- Run `modinfo bluetooth` and find the `depends` field — see which other modules it relies on
- Run `sysctl net.ipv4.ip_forward` and check the current value — note what it would mean to change it

---

*Up next: Module 16 — Automation & Scheduled Jobs*
