# Linux Basics for Hackers
## Module 1 - The Basics of the Terminal

---

## Overview

In Windows you click things. In Linux you type things. That's the shift this module is about. The terminal is where you'll spend most of your time as a hacker, and every tool you'll eventually use runs through it. There's no skipping this.

---

## What the terminal actually is

The terminal is a text window where you give the computer direct instructions and it responds. No icons, no menus, no mouse required. Just you typing and the system doing exactly what you said.

Most hacking tools have no graphical interface at all - no buttons, no windows, nothing to click. If you can't use the terminal, you can't use the tools. It's that simple.

It's worth thinking of the terminal as a conversation with your computer. Instead of hunting for a folder and double-clicking it, you just type where you want to go and you're there instantly. It feels slow at first and then one day it feels faster than any GUI you've used.

---

## The Linux filesystem - the upside-down tree

Linux stores everything in a single structure that branches out from one starting point at the top. That starting point is called **root**, and it's written as `/` - just a forward slash.

Everything on the system - files, folders, programs, settings - lives somewhere inside that tree.

![Linux Filesystem Hierarchy](assets/linux_filesystem_tree_1789211965428.jpg)

```
/
├── etc/        system settings and config files
├── bin/        essential built-in programs
├── home/       personal folders for each user
│   └── kali/   your home folder
├── root/       home folder for the root (admin) user
├── var/        logs and data that changes constantly
├── tmp/        temporary files, wiped on every reboot
└── usr/        installed software and applications
```

If you're coming from Windows: Windows uses `C:\Users\YourName\` as your home. Linux uses `/home/yourname/`. Same idea, different syntax.

The important folders to remember right now:

| Folder | What's inside |
|---|---|
| `/` | The root - the top of everything |
| `/etc` | Config files - system settings live here |
| `/bin` | Basic programs the system needs to run |
| `/home` | Personal folders for regular users |
| `/root` | Home folder for the root user specifically |
| `/tmp` | Temporary files - cleared every reboot |

---

## Essential Commands

### pwd

Stands for "Print Working Directory." Tells you exactly where you are in the filesystem right now.

```bash
ahegazy0@kali:~$ pwd
/home/kali
```

You're in the `kali` folder, inside `home`, at the root of the system. Use this whenever you feel lost.

---

### ls

Stands for "List." Shows you what's inside your current folder.

```bash
ahegazy0@kali:~$ ls
Desktop  Documents  Downloads  Pictures
```

The useful variations:

| Command | What changes |
|---|---|
| `ls` | Basic list |
| `ls -l` | Detailed list with permissions, size, and date |
| `ls -a` | Shows hidden files too |
| `ls -la` | Detailed list including hidden files |

Hidden files in Linux start with a dot - like `.bashrc` or `.config`. They don't show up with a plain `ls`. Use `ls -la` when you need to see everything, and you'll use it often.

---

### cd

Stands for "Change Directory." This is how you move around.

```bash
ahegazy0@kali:~$ cd /etc
```

Goes straight to `/etc`.

The shortcuts worth memorizing:

| Command | Where it takes you |
|---|---|
| `cd /path` | Straight to whatever path you type |
| `cd ..` | Up one level to the parent folder |
| `cd ~` | Straight to your home folder |
| `cd /` | All the way to the root |
| `cd -` | Back to the previous folder you were in |

`cd ..` is the one you'll use constantly. If you're deep in the tree and want to climb back up, keep typing `cd ..` until you're where you want to be.

---

### man

Stands for "Manual." Every command in Linux has a built-in manual page that explains what it does and lists every option available.

```bash
ahegazy0@kali:~$ man ls
```

Opens the full manual for `ls`.

Inside the manual:
- Scroll down with `j` or the down arrow
- Scroll up with `k` or the up arrow
- Search for a word by pressing `/` then typing it
- Quit by pressing `q`

When you encounter a tool you've never used before, `man toolname` is always the first thing to run. The answer is usually in there.

---

### whoami

Shows you which user you're currently logged in as.

```bash
ahegazy0@kali:~$ whoami
kali
```

Or if you're running as the admin:
```
root
```

This matters a lot later. Some commands require root access and won't work as a regular user. Knowing what account you're on saves a lot of confusion.

---

### locate

Searches the entire filesystem for a file by name.

```bash
ahegazy0@kali:~$ locate nmap
```

Finds every file on the system with "nmap" in its name, instantly.

One thing to know: `locate` works from a pre-built database, not by scanning the disk in real time. If you installed something recently and `locate` can't find it, update the database first:

```bash
ahegazy0@kali:~$ updatedb
```

This requires root access. Run it once and then `locate` will be up to date.

> **Gotcha:** `locate` won't find files in your `/root/` directory unless you run it as the root user. If you are a normal user, it pretends those files don't exist.

---

## The one rule you can't forget

**Linux is case-sensitive.** This trips up almost every beginner.

```bash
ahegazy0@kali:~$ cd Desktop     works
ahegazy0@kali:~$ cd desktop     error - no such file
ahegazy0@kali:~$ cd DESKTOP     error - no such file
```

`Desktop`, `desktop`, and `DESKTOP` are three completely different things as far as Linux is concerned. Always watch your capitalization.

---

## How navigation looks in practice

```
/
└── home/
    └── kali/
        ├── Desktop/
        └── Downloads/

You're at /home/kali. You want to get to Desktop:
  cd Desktop             one step down
  cd ..                  back to /home/kali
  cd ../..               back to /home
  cd /                   jump straight to root from anywhere
  cd ~                   jump straight to /home/kali from anywhere
```

---

## Command Reference

| Command | What it does | Example |
|---|---|---|
| `pwd` | Shows current location | `pwd` |
| `ls` | Lists files in current folder | `ls` |
| `ls -la` | Lists everything including hidden files | `ls -la` |
| `cd /path` | Go to a folder | `cd /etc` |
| `cd ..` | Go up one level | `cd ..` |
| `cd ~` | Go to home folder | `cd ~` |
| `man cmd` | Open the manual for a command | `man ls` |
| `whoami` | Shows which user you are | `whoami` |
| `locate file` | Find a file anywhere on the system | `locate nmap` |
| `updatedb` | Refresh the locate database | `updatedb` |

---

## Practice

- [ ] Open the terminal and run `pwd` - see where you start
- [ ] Run `cd /` then `ls` - you're at the root, look at what's there
- [ ] Run `cd ~` then `ls -la` - spot the hidden files starting with `.`
- [ ] Run `man nmap`, scroll through it, and exit with `q`

The goal isn't to memorize all of this right now. It's to get your hands moving and start feeling comfortable in the terminal. The commands will stick on their own the more you use them.

> 💡 *For deeper practice, I also recommend completing the end-of-chapter exercises in the official **Linux Basics for Hackers** book.*
---

*Up next: Module 2 - Text Manipulation*
