# Linux Basics for Hackers
## Module 2 - Text Manipulation

---

## Overview

In Linux, almost everything is a text file. System settings, logs, configs, user data - all text. Once you know how to search through and manipulate text, you can dig through thousands of lines of output and pull exactly what you need in seconds. That's the whole point of this module.

---

## Why this matters for hacking

When a scanning tool runs and produces output, it doesn't give you a clean summary. It dumps everything - thousands of lines. You're not reading that manually. You use text tools to filter it down to only what you care about. A real example: after scanning a network, you'd use `grep` to pull only the lines containing "open" to find open ports, ignoring everything else.

Also, nearly every system configuration lives in a plain text file somewhere under `/etc`. If you can read and edit those files, you can reconfigure almost anything on the system.

---

## Essential Commands

### cat

Short for "concatenate." The simplest way to read a file - it just dumps the entire contents to the screen.

```bash
ahegazy0@kali:~$ cat /etc/passwd
```

> **Gotcha:** Never run `cat` on a compiled binary file (like `cat /bin/ls`). It dumps raw bytecode to your screen and scrambles your terminal font into unreadable alien symbols. If this happens, type `reset` blindly and hit Enter.

Also useful for creating small files quickly:

```bash
ahegazy0@kali:~$ cat > targets.txt
```

After running that, type your content line by line. When done, press `Ctrl+D` to save and exit. Whatever you typed is now in `targets.txt`.

To add content to an existing file without overwriting it, use `>>` instead:

```bash
ahegazy0@kali:~$ cat >> targets.txt
```

The difference between `>` and `>>` is important. `>` overwrites. `>>` appends.

---

### grep

This is the one you'll use the most. `grep` searches through a file and returns only the lines that contain a specific word or pattern.

```bash
ahegazy0@kali:~$ grep "password" logs.txt
```

Returns every line in `logs.txt` that contains the word "password."

Useful options:

| Option | What it does |
|---|---|
| `-i` | Case-insensitive search (finds "Password", "PASSWORD", etc.) |
| `-r` | Search recursively through all files in a folder |
| `-n` | Show line numbers alongside results |
| `-v` | Invert - show lines that do NOT contain the word |

Example:
```bash
ahegazy0@kali:~$ grep -i "admin" access.log
```

Finds "admin", "Admin", "ADMIN" - all of them.

---

### head and tail

When a file is huge and you only want a small piece of it:

```bash
ahegazy0@kali:~$ head /etc/snort/snort.conf
```

Shows the first 10 lines by default.

```bash
ahegazy0@kali:~$ tail /var/log/syslog
```

Shows the last 10 lines by default.

To control how many lines you get:

```bash
ahegazy0@kali:~$ head -n 20 file.txt     first 20 lines
ahegazy0@kali:~$ tail -n 20 file.txt     last 20 lines
```

`tail` has one especially useful trick - the `-f` flag, which follows a file in real time:

```bash
ahegazy0@kali:~$ tail -f /var/log/syslog
```

This keeps the terminal open and shows new lines as they're added to the file. Useful for watching logs live while something is running.

---

### nl

Adds line numbers to a file's output. Handy when you need to reference specific lines.

```bash
ahegazy0@kali:~$ nl /etc/snort/snort.conf
```

---

### less

When a file is too long to read with `cat` (it just blasts past you), use `less` instead. It lets you scroll through the file one page at a time.

```bash
ahegazy0@kali:~$ less /etc/snort/snort.conf
```

Controls:
- `Space` - next page
- `b` - previous page
- `/word` - search for a word
- `q` - quit

---

### sed

Short for "stream editor." It finds a word or pattern in a file and replaces it with something else.

```bash
ahegazy0@kali:~$ sed 's/mysql/MySQL/g' config.txt
```

Breaking that down:
- `s/` - substitute
- `mysql` - find this
- `/MySQL/` - replace with this
- `g` - do it globally (every occurrence, not just the first)

By default, `sed` only prints the modified output - it doesn't actually change the file. To save the changes back to the file, add `-i`:

```bash
ahegazy0@kali:~$ sed -i 's/mysql/MySQL/g' config.txt
```

> Before using `-i`, make a backup. `sed -i` modifies files permanently and there's no undo.

```bash
ahegazy0@kali:~$ cp config.txt config.txt.bak
ahegazy0@kali:~$ sed -i 's/old/new/g' config.txt
```

---

## The pipe - connecting commands together

The pipe `|` takes the output of one command and feeds it directly into another. This is where things get powerful.

![Linux Pipeline Diagram](assets/linux_pipe_diagram_1789213616381.jpg)

```bash
ahegazy0@kali:~$ cat access.log | grep "failed"
```

Instead of reading the whole log, you only see the lines containing "failed."

You can chain as many pipes as you want:

```bash
ahegazy0@kali:~$ cat access.log | grep "failed" | tail -n 20
```

That reads the log, filters for "failed", then shows only the last 20 of those filtered lines. Three commands, one line.

---

## Command Reference

| Command | What it does | Example |
|---|---|---|
| `cat file` | Print entire file to screen | `cat /etc/passwd` |
| `cat > file` | Create a file and type into it | `cat > notes.txt` |
| `grep "word" file` | Find lines containing a word | `grep "root" passwd` |
| `grep -i` | Case-insensitive search | `grep -i "admin" log` |
| `head -n 20 file` | First 20 lines | `head -n 20 file.txt` |
| `tail -n 20 file` | Last 20 lines | `tail -n 20 file.txt` |
| `tail -f file` | Follow file in real time | `tail -f syslog` |
| `nl file` | Show file with line numbers | `nl config.conf` |
| `less file` | Scroll through a file | `less bigfile.txt` |
| `sed 's/a/b/g' file` | Replace all "a" with "b" | `sed 's/old/new/g' f` |
| `cmd1 \| cmd2` | Pipe output of cmd1 into cmd2 | `cat log \| grep fail` |

---

## Practice

- [ ] Go to `/etc/snort/` and open `snort.conf` with `less` - scroll through it to get a feel for what a config file looks like
- [ ] Use `grep "output" /etc/snort/snort.conf` and see which lines come back
- [ ] Use `cat > targets.txt`, type three IP addresses (one per line), then press `Ctrl+D` - then read it back with `cat targets.txt`
- [ ] Try chaining: `cat /etc/snort/snort.conf | grep "output" | nl`

> 💡 *For deeper practice, I also recommend completing the end-of-chapter exercises in the official **Linux Basics for Hackers** book.*
---

*Up next: Module 3 - Managing Networks*
