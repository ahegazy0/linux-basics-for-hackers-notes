# Linux Basics for Hackers
## Module 4 - Software Management

---

## Overview

Kali comes loaded with tools but you'll constantly need to add new ones, update existing ones, or pull something directly from a developer's GitHub. This module covers how Linux handles software installation - which works nothing like Windows - and how to get tools that aren't in the standard library.

---

## How Linux handles software

On Windows, you download an installer from a website, double-click it, and click through the wizard. Linux doesn't work like that.

Linux uses **repositories** - massive online libraries of software maintained and verified by the distribution. Instead of hunting for a download link, you just tell your system the name of what you want and it finds it, downloads it, and installs it automatically, including any other software it depends on to run.

The tool that manages all of this on Kali (and Debian-based systems) is `apt-get`.

The list of repositories your system knows about lives in a file called `sources.list` at `/etc/apt/sources.list`. You generally leave this file alone unless you know what you're adding - unofficial repositories can contain malicious packages.

---

## Essential Commands

### apt-get update

Before installing anything, run this. It doesn't install or upgrade anything - it just refreshes your local list of what's available in the repositories. If you skip this and try to install something, you might get an old version or an error.

```bash
ahegazy0@kali:~$ apt-get update
```

Make a habit of running this before any install.

---

### apt-get install

Downloads and installs a package. Kali handles dependencies automatically - if the tool needs five other libraries to run, it grabs those too.

```bash
ahegazy0@kali:~$ apt-get install wireshark
```

It'll ask for confirmation before downloading. Type `y` and press Enter.

To skip the confirmation prompt:

```bash
ahegazy0@kali:~$ apt-get install -y wireshark
```

The `-y` flag answers yes automatically. Useful when you know what you're doing.

---

### apt-get remove

Removes an installed package.

```bash
ahegazy0@kali:~$ apt-get remove wireshark
```

This removes the program but leaves behind its configuration files. If you want to remove everything including configs:

```bash
ahegazy0@kali:~$ apt-get purge wireshark
```

---

### apt-get upgrade

Updates all installed packages to their latest versions.

```bash
ahegazy0@kali:~$ apt-get upgrade
```

Good to run periodically to keep your tools current. Always run `apt-get update` first to refresh the package list before upgrading.

---

### apt-cache search

Searches the local repository database for packages matching a keyword. Useful when you know roughly what you're looking for but not the exact package name.

```bash
ahegazy0@kali:~$ apt-cache search wifi
```

Returns a list of packages with "wifi" in their name or description. From there you pick what looks right and install it by name.

```bash
ahegazy0@kali:~$ apt-cache search wireless
ahegazy0@kali:~$ apt-cache search password crack
```

---

### git clone

A lot of the best and newest hacking tools aren't in any repository - they live on GitHub. `git clone` downloads a copy of the entire project from GitHub directly to your machine.

```bash
ahegazy0@kali:~$ git clone https://github.com/username/toolname
```

This creates a folder with the tool's name in your current directory. Most tools come with a `README` file that explains how to install and run them from there.

The general process after cloning:

```bash
ahegazy0@kali:~$ git clone https://github.com/example/tool
ahegazy0@kali:~$ cd tool
```

Then read the README for install steps - usually something like `pip install -r requirements.txt` or just running a Python script directly.

> Many of the most current tools only exist on GitHub. Learning `git clone` early means you're not limited to what's in the official repositories.

---

## A note on sources.list

The file `/etc/apt/sources.list` contains the URLs of all the repositories your system pulls from. You can open it with:

```bash
ahegazy0@kali:~$ cat /etc/apt/sources.list
```

Sometimes you'll find instructions online telling you to add a line to this file to access a third-party repository. Be careful with that. Official Kali repositories are maintained and verified. Random third-party repos are not. Adding the wrong source is an easy way to install something you didn't intend to.

---

## Command Reference

| Action | Command |
|---|---|
| Refresh package list | `apt-get update` |
| Install a package | `apt-get install [name]` |
| Install without prompt | `apt-get install -y [name]` |
| Remove a package | `apt-get remove [name]` |
| Remove including configs | `apt-get purge [name]` |
| Update all packages | `apt-get upgrade` |
| Search for a package | `apt-cache search [keyword]` |
| Clone from GitHub | `git clone [url]` |

---

## Practice

- [ ] Run `apt-get update` to refresh your package list
- [ ] Use `apt-cache search wifi` and look through what comes back
- [ ] Pick one tool from the results and install it with `apt-get install`
- [ ] Find any simple tool on GitHub and bring it to your machine with `git clone`

> 💡 *For deeper practice, I also recommend completing the end-of-chapter exercises in the official **Linux Basics for Hackers** book.*
---

*Up next: Module 5 - Controlling File and Directory Permissions*
