# Linux Basics for Hackers
## Module 16 — Automation & Scheduled Jobs

---

## What this module is about

Typing the same commands manually every day is a waste of time. Linux has a built-in scheduling system called cron that lets you tell the system to run any script or command at any time, on any schedule you define. This module covers how to set that up, how to make things run at startup, and how the whole automation system works.

---

## Cron — the Linux scheduler

Cron is a background service that wakes up every minute, checks if any scheduled jobs are due to run, and executes them. It's been part of Unix systems for decades and is still the standard way to automate tasks on Linux.

Each user has their own cron schedule called a **crontab** (cron table). Root has one. You have one. They're separate, and they run with the permissions of the user they belong to.

---

## The crontab syntax

Every line in a crontab represents one scheduled job and follows this format:

```
MIN  HOUR  DAY  MONTH  WEEKDAY  command
```

Five time fields, then the command to run. The fields:

| Field | Range | What it means |
|---|---|---|
| MIN | 0–59 | Minute of the hour |
| HOUR | 0–23 | Hour of the day (24-hour clock) |
| DAY | 1–31 | Day of the month |
| MONTH | 1–12 | Month of the year |
| WEEKDAY | 0–7 | Day of the week (0 and 7 are both Sunday) |

A `*` in any field means "every" — every minute, every hour, every day, etc.

Some examples to make it concrete:

```bash
# Every minute, every hour, every day
* * * * *  /path/to/script.sh

# Every day at 3:00 AM
0 3 * * *  /path/to/script.sh

# Every Wednesday at 3:00 AM
0 3 * * 3  /path/to/script.sh

# The 15th of every month at noon
0 12 15 * *  /path/to/script.sh

# Every Monday through Friday at 8:00 AM
0 8 * * 1-5  /path/to/script.sh

# Every hour on the hour
0 * * * *  /path/to/script.sh
```

The way to read them: fill in left to right. `0 3 * * 3` — at minute 0, hour 3, any day of the month, any month, but only on weekday 3 (Wednesday).

---

## Editing your crontab

```bash
crontab -e
```

This opens your personal crontab in a text editor. Add one job per line. Save and close — cron picks up the changes immediately.

**View your current crontab:**

```bash
crontab -l
```

**Remove all your cron jobs:**

```bash
crontab -r
```

Be careful with `-r` — there's no confirmation prompt and no undo.

**Edit root's crontab** (requires sudo):

```bash
sudo crontab -e
```

Jobs in root's crontab run with full root permissions.

---

## Shortcuts — special cron syntax

Instead of the five-field format, cron supports some shorthand:

| Shorthand | Equivalent to | When it runs |
|---|---|---|
| `@reboot` | — | Once, at system startup |
| `@hourly` | `0 * * * *` | Every hour |
| `@daily` | `0 0 * * *` | Every day at midnight |
| `@weekly` | `0 0 * * 0` | Every Sunday at midnight |
| `@monthly` | `0 0 1 * *` | First day of every month |

`@reboot` is particularly useful — it runs your command once every time the system boots, regardless of time.

```bash
@reboot  /home/kali/scripts/startup.sh
```

---

## A practical example

Write a script that logs the current date and time to a file:

```bash
#!/bin/bash
echo "System check: $(date)" >> /home/kali/check.log
```

Save it as `check.sh`, make it executable:

```bash
chmod 755 /home/kali/check.sh
```

Schedule it to run every minute to test it:

```bash
crontab -e
```

Add:
```
* * * * *  /home/kali/check.sh
```

Save, wait a couple of minutes, then check:

```bash
cat /home/kali/check.log
```

You should see new entries appearing. Once confirmed, change the schedule to whatever you actually want.

---

## Cron output and logging

By default, cron emails you the output of any job that produces output. On a local system with no mail configured, that output often just goes nowhere. To redirect it yourself:

```bash
* * * * *  /home/kali/check.sh >> /home/kali/cron.log 2>&1
```

`>> /home/kali/cron.log` appends stdout to the log file.
`2>&1` redirects stderr to the same place, so errors show up in the log too.

If you want the job to run silently with no output at all:

```bash
* * * * *  /home/kali/check.sh > /dev/null 2>&1
```

`/dev/null` is a special file that discards everything written to it.

---

## Making services start at boot

For services rather than scripts, `systemctl enable` is the standard modern approach:

```bash
systemctl enable mysql
systemctl enable apache2
systemctl enable ssh
```

This creates the necessary symlinks so that systemd (the system and service manager) starts those services automatically during boot.

Check what's currently enabled:

```bash
systemctl list-unit-files --type=service | grep enabled
```

**The older method — update-rc.d:**

On older Debian/Ubuntu-based systems you may see `update-rc.d` used instead:

```bash
update-rc.d mysql defaults
```

This registers a service with the old init.d startup system. `systemctl enable` does the same thing on modern systems and is what you should use going forward.

---

## The /etc/cron directories

Alongside user crontabs, the system has its own scheduled directories:

```
/etc/cron.hourly/     scripts here run every hour
/etc/cron.daily/      scripts here run every day
/etc/cron.weekly/     scripts here run every week
/etc/cron.monthly/    scripts here run every month
```

Drop an executable script into any of these folders and it gets run on that schedule automatically — no crontab entry needed. System maintenance tasks and log rotation often work this way.

---

## Command reference

| Command | What it does |
|---|---|
| `crontab -e` | Edit your personal cron schedule |
| `crontab -l` | View your current cron jobs |
| `crontab -r` | Delete all your cron jobs |
| `sudo crontab -e` | Edit root's cron schedule |
| `systemctl enable [service]` | Start a service automatically at boot |
| `systemctl disable [service]` | Remove a service from auto-start |
| `systemctl list-unit-files --type=service` | See all services and their startup status |

---

## Practice

- Open your crontab with `crontab -e` and add a job that writes the current date to a file every minute — verify it works after two minutes, then remove the job
- Schedule a script to run at `@reboot` and confirm it executes after a reboot
- Run `crontab -l` on root and see if there are any system jobs already scheduled
- Enable MySQL to start at boot with `systemctl enable mysql`, then reboot and check if it's running with `systemctl status mysql`
- Try to write the cron syntax for: every day at 6:30 AM, only on weekdays — work it out before checking the answer

The cron syntax takes a few tries to get comfortable with. Writing a few different schedules from scratch is the fastest way to internalize it.

---

*Up next: Module 17 — Python Scripting*
