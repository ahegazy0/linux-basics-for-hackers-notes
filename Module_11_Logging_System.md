# Linux Basics for Hackers
## Module 11 — The Logging System

---

## What this module is about

Linux writes down almost everything that happens on the system — logins, failed logins, hardware events, service activity, errors. These log files are useful for two completely opposite reasons depending on which side of the keyboard you're on. As a defender, they tell you if someone is poking around. As an attacker, they're a record of everything you did and need to be dealt with before you leave.

---

## Where logs live

Almost all logs are stored in `/var/log`. Navigate there and list the contents:

```bash
ls /var/log
```

You'll see a lot of files. The important ones to know:

| Log file | What it records |
|---|---|
| `/var/log/auth.log` | Login attempts, sudo usage, SSH connections |
| `/var/log/syslog` | General system messages — catch-all for most events |
| `/var/log/kern.log` | Kernel messages, hardware events, driver errors |
| `/var/log/messages` | Similar to syslog, used on some distros instead |
| `/var/log/mail.log` | Email server activity |
| `/var/log/apache2/` | Web server access and error logs (if Apache is running) |
| `/var/log/mysql/` | Database activity (if MySQL is running) |

The one you'll look at most is `auth.log`. It shows every login attempt — successful or not — along with the source IP if it came in over the network.

---

## Reading logs

Logs are plain text files. You can read them with any text tool.

**See the last 10 lines:**

```bash
tail /var/log/auth.log
```

**Follow a log in real time** (watch it update as new entries come in):

```bash
tail -f /var/log/auth.log
```

The `-f` flag means "follow." This is useful when you're watching for something specific — like monitoring login attempts while someone is actively trying to get in.

**Search a log for something specific:**

```bash
grep "Failed" /var/log/auth.log
```

This pulls out every line that contains the word "Failed" — so you're left with just the failed login attempts, not the noise of everything else.

```bash
grep "Failed" /var/log/auth.log | grep "192.168.1.50"
```

Now you're filtering for failed logins from a specific IP. This is how you check if one machine is hammering yours with password attempts.

---

## rsyslog — what actually writes the logs

The background service responsible for collecting and writing log data is called **rsyslog**. It runs silently at all times, listening for messages from the kernel and other programs, and writing them to the appropriate files in `/var/log`.

You can check if it's running:

```bash
service rsyslog status
```

And stop it:

```bash
service rsyslog stop
```

Stopping rsyslog means the system stops writing new log entries. Anything that happens after that won't be recorded. This is a tactic, but it's a crude one — any competent admin monitoring the system will notice that logging has gone dark. Log gaps are their own kind of red flag.

---

## logrotate — keeping logs from filling the disk

Logs never stop growing. If nothing managed them, `/var/log` would eventually fill the entire disk. `logrotate` is the tool that handles this automatically.

It works on a schedule (usually daily) and does the following:
- Renames the current log file (e.g., `auth.log` becomes `auth.log.1`)
- Creates a fresh empty `auth.log` for new entries
- Compresses older rotated files (`auth.log.2.gz`, etc.)
- Deletes logs older than a configured number of days

The configuration that controls all of this lives in:

```
/etc/logrotate.conf          ← main config
/etc/logrotate.d/            ← per-service configs (apache, mysql, etc.)
```

Opening `/etc/logrotate.conf` shows you things like how many old log versions to keep and how often rotation runs. This is the answer to the practice challenge in your notes about finding where rotation is configured.

---

## shred — destroying files properly

When you delete a file normally with `rm`, the data doesn't actually disappear. The filesystem just marks that space as available for reuse. Until something else overwrites it, the original data is still physically on the disk and can be recovered with forensic tools.

`shred` overwrites a file multiple times with random data before deleting it, making recovery practically impossible.

```bash
shred -vzu filename.txt
```

Breaking down those flags:
- `-v` — verbose, shows progress
- `-z` — after shredding, does one final overwrite with zeros to hide that shredding happened
- `-u` — removes the file after shredding

By default shred overwrites 3 times. You can increase that:

```bash
shred -n 10 -vzu filename.txt
```

This overwrites 10 times before zeroing and deleting.

**On a log file:**

```bash
shred -vzu /var/log/auth.log
```

After this, even if someone recovers the file, they get random data. The log is gone.

---

## The smarter approach — editing logs, not deleting them

Deleting an entire log file is obvious. A syslog that suddenly has no `auth.log` at all raises immediate suspicion — almost as much as logs going completely silent.

What experienced attackers do instead is open the log file, find the lines that contain their activity (their IP address, their username, the timestamp of their session), remove only those lines, and save the file. The log still exists, still has content, still looks normal — but their specific entries are gone.

To find your entries:

```bash
grep "192.168.1.100" /var/log/auth.log
```

Then open the file in a text editor like `nano` or `vi`, locate those lines, delete them, and save. The log file remains intact and unsuspicious — just with a small invisible gap where you were.

This is more effort than `shred`, but it's far less likely to be noticed.

---

## Covering tracks — the full picture

From an attacker's perspective, the log cleanup checklist usually looks something like this:

1. Check which logs recorded your activity — `auth.log` for logins, service-specific logs if you interacted with a web server or database
2. Decide whether to edit (surgical, less suspicious) or shred (complete, but obvious if someone looks)
3. Clear your bash history for the session — `export HISTSIZE=0` and `history -c`
4. Check if any other tools logged your actions (IDS, web server access logs, database query logs)

Logs are spread across the system. Missing one is common. That's why forensic investigators look in multiple places, not just `/var/log/auth.log`.

---

## From a defensive perspective

If you're securing a system rather than attacking one:

- `tail -f /var/log/auth.log` is something you should have open if you suspect someone is trying to get in
- Repeated "Failed password" entries from the same IP is a brute force attempt — block that IP with `iptables` or `ufw`
- A log file that's completely empty, missing, or recently shredded is itself evidence of tampering
- Tools like `fail2ban` can automatically block IPs that fail login too many times — worth setting up on any internet-facing machine

---

## Command reference

| Command | What it does |
|---|---|
| `ls /var/log` | See all log files |
| `tail /var/log/auth.log` | Last 10 lines of auth log |
| `tail -f /var/log/auth.log` | Follow auth log in real time |
| `grep "term" /var/log/file` | Search a log for a specific string |
| `service rsyslog status` | Check if logging service is running |
| `service rsyslog stop` | Stop the logging service |
| `shred -vzu filename` | Overwrite and delete a file securely |
| `shred -n 10 -vzu filename` | Same but with 10 overwrite passes |
| `history -c` | Clear your bash command history |
| `cat /etc/logrotate.conf` | See the log rotation configuration |

---

## Practice

- Navigate to `/var/log` and run `ls` to see what's there
- Run `tail /var/log/auth.log` and look at the last few entries — you should see your own recent login
- Run `tail -f /var/log/auth.log` in one terminal, then open another terminal and try `ssh localhost` — watch the entry appear in real time
- Create a dummy file with `echo "sensitive data" > test.txt`, then run `shred -vzu test.txt` and try to find the data afterward
- Open `/etc/logrotate.conf` and find the line that controls how many old log versions are kept

---

*Up next: Module 12 — Using & Abusing Services*
