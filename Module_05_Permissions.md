# Linux Basics for Hackers
## Module 5 - Permissions & Privileges

---

## Overview

Linux is a multi-user system. Multiple people can have accounts on the same machine, and not everyone should be able to read, modify, or run everything. Permissions are how Linux enforces that. This module covers how to read permissions, change them, and understand where they become a security issue.

---

## How permissions work

Every file and folder in Linux has three permission groups attached to it:

- **Owner** - the user who created the file
- **Group** - a collection of users who share access
- **Others** - everyone else on the system

Each of those three groups can have three types of permission:

| Permission | Symbol | What it means |
|---|---|---|
| Read | `r` | Can view the file's contents |
| Write | `w` | Can modify or delete the file |
| Execute | `x` | Can run the file as a program |

![Linux File Permissions](assets/file_permissions_diagram_1789213222578.jpg)

When you run `ls -l`, the first column shows you the permissions for every file:

```bash
ahegazy0@kali:~$ ls -l
-rwxr-xr--  1  kali  kali  4096  Jan 1  file.sh
```

That string of characters at the start is the permission block. Breaking it down:

```
- rwx r-x r--
│  │   │   │
│  │   │   └── Others:  read only
│  │   └────── Group:   read + execute
│  └────────── Owner:   read + write + execute
└───────────── File type: - means regular file, d means directory
```

If you download a hacking tool and it refuses to run, this is usually why - the execute bit isn't set.

---

## chmod - changing permissions

`chmod` stands for "change mode." It's how you set who can do what with a file.

There are two ways to use it: the numeric method and the symbolic method.

### Numeric method

Each permission has a number value:

| Permission | Value |
|---|---|
| Read (r) | 4 |
| Write (w) | 2 |
| Execute (x) | 1 |

You add them together to get the permission for each group. Then you write three digits - one for owner, one for group, one for others.

| Number | Calculation | Permissions |
|---|---|---|
| 7 | 4+2+1 | Read + Write + Execute |
| 6 | 4+2 | Read + Write |
| 5 | 4+1 | Read + Execute |
| 4 | 4 | Read only |
| 0 | 0 | No permissions at all |

Examples:

```bash
ahegazy0@kali:~$ chmod 755 script.sh
```

Owner gets 7 (read+write+execute). Group gets 5 (read+execute). Others get 5 (read+execute). This is the standard permission for a script you want to run but not let others modify.

```bash
ahegazy0@kali:~$ chmod 644 notes.txt
```

Owner gets 6 (read+write). Group gets 4 (read). Others get 4 (read). Standard for a file that others can read but only you can edit.

```bash
ahegazy0@kali:~$ chmod 777 file.sh
```

> **Gotcha:** `chmod 777` is the ultimate lazy solution for a script that won't run, but it means *anyone* on the system can modify your file. In a pentesting exam or a real environment, blindly changing a sensitive file to 777 is an immediate security failure. Get comfortable using `chmod +x` instead!

### Symbolic method

Instead of numbers, you use letters to specify changes:

```bash
ahegazy0@kali:~$ chmod +x script.sh
```

Adds execute permission for everyone.

```bash
ahegazy0@kali:~$ chmod u+x script.sh
```

Adds execute only for the owner (`u` = user/owner).

```bash
ahegazy0@kali:~$ chmod g-w file.txt
```

Removes write permission from the group.

The letters: `u` = owner, `g` = group, `o` = others, `a` = all three.

---

## chown - changing ownership

`chown` stands for "change owner." It reassigns who owns a file.

```bash
ahegazy0@kali:~$ chown kali file.txt
```

Changes the owner to the user `kali`.

```bash
ahegazy0@kali:~$ chown kali:kali file.txt
```

Changes both the owner and the group to `kali`.

```bash
ahegazy0@kali:~$ chown -R kali /home/kali/tools
```

The `-R` flag applies the change recursively - it changes ownership on the folder and everything inside it.

You need root access to change ownership of files you don't own.

---

## SUID - the important one

SUID stands for "Set User ID." It's a special permission bit that changes how a file executes.

Normally when you run a program, it runs with your permissions. If the SUID bit is set on a file owned by root, it runs with root's permissions - even if you're a regular user.

In the permissions string, SUID shows up as an `s` where the owner's execute bit would be:

```
-rwsr-xr-x   root   root   /usr/bin/passwd
```

That `s` in the owner field means SUID is set. The `passwd` command needs SUID because it has to write to `/etc/shadow` (a root-owned file) when any user changes their password.

SUID is legitimate when used intentionally. The problem is when files have SUID set by mistake or when poorly written SUID programs can be tricked into doing things they shouldn't.

**This is where privilege escalation comes in.** Privilege escalation is the process of going from a limited user account to root by exploiting permission mistakes. Finding a misconfigured SUID binary is one of the classic ways to do it.

To find all files on the system with the SUID bit set:

```bash
ahegazy0@kali:~$ find / -perm -u=s -type f 2>/dev/null
```

Breaking that down:
- `find /` - search from root
- `-perm -u=s` - find files with SUID bit set
- `-type f` - files only, not directories
- `2>/dev/null` - hide permission errors so the output is clean

Run that on any Linux system and look through the results. Any file there that has a known vulnerability or misconfiguration is a potential escalation path.

---

## A quick word on privilege escalation

The whole point of understanding permissions deeply is this: systems get compromised at the user level all the time. An attacker gets in as a low-privilege user and then looks for ways to gain root. That process is privilege escalation.

Permission misconfigurations are one of the most common vectors - SUID binaries that shouldn't be SUID, files writable by the wrong users, scripts owned by root but editable by everyone. Knowing how permissions work is what lets you spot those mistakes, whether you're attacking or defending.

---

## Command Reference

| Command | What it does | Example |
|---|---|---|
| `ls -l` | Show permissions for files | `ls -l` |
| `chmod 755 file` | Set permissions numerically | `chmod 755 script.sh` |
| `chmod +x file` | Add execute for everyone | `chmod +x tool.py` |
| `chmod u+x file` | Add execute for owner only | `chmod u+x run.sh` |
| `chown user file` | Change file owner | `chown kali file.txt` |
| `chown user:group file` | Change owner and group | `chown kali:kali file` |
| `chown -R user dir` | Recursive ownership change | `chown -R kali /tools` |
| `find / -perm -u=s -type f 2>/dev/null` | Find all SUID files | - |

---

## Practice

- [ ] Create a file with `touch test.txt` then run `ls -l` to read its default permissions
- [ ] Use `chmod 600 test.txt` - that gives only you read and write, nobody else anything. Confirm with `ls -l`
- [ ] Create a simple script, try to run it, watch it fail, then use `chmod +x` and run it again
- [ ] Run the `find` command above to list all SUID files on your system - look through what comes back

> 💡 *For deeper practice, I also recommend completing the end-of-chapter exercises in the official **Linux Basics for Hackers** book.*
---

*Up next: Module 6 - Process Management*
