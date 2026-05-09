# Linux Basics for Hackers
## Module 6 — Process Management

---

## What this module is about

At any given moment, your computer is running hundreds of things at once — your terminal, background services, system daemons, things you didn't even knowingly start. This module is about seeing all of that, understanding what's what, and being able to control it. That means speeding things up, moving tasks around, and killing processes that are frozen, eating resources, or just in your way.

---

## What is a process?

Every time you run a program, the system creates a **process** for it. A process is just a running instance of a program. Open Firefox — that's a process. Run a terminal command — that's a process. Even things running silently in the background with no visible window are processes.

The key thing to understand is that every process gets a unique number assigned to it the moment it starts. That number is called the **PID — Process ID**. It's how the system (and you) identify and refer to a specific running task. If you want to stop something, you need its PID.

Two types of processes worth knowing:

- **Foreground processes** — running visibly, attached to your terminal. If you close the terminal, they die.
- **Background processes** — running silently behind the scenes. Your terminal stays free while they run.

---

## Seeing what's running

### ps

`ps` gives you a snapshot of the processes running right now. On its own it only shows processes tied to your current terminal session, which isn't very useful. The version you actually want is:

```bash
ps aux
```

Breaking down those flags:
- `a` — show processes from all users, not just you
- `u` — show the user who owns each process
- `x` — include processes not attached to any terminal (background daemons)

The output looks like this:

```
USER       PID  %CPU  %MEM    VSZ   RSS  STAT  COMMAND
root         1   0.0   0.1  22548  1024  Ss    /sbin/init
kali      1023   0.3   1.2  54320  6200  S     bash
kali      1587   2.1   4.5 412300 22800  Sl    firefox
```

The columns that matter most:

| Column | What it tells you |
|---|---|
| USER | Who owns the process |
| PID | The process ID — this is what you'll use to kill it |
| %CPU | How much CPU it's using |
| %MEM | How much RAM it's using |
| COMMAND | What program is actually running |

`ps aux` is a static snapshot. It shows you the state of things at the exact moment you ran the command, then stops updating.

---

### top

`top` is the live version. It refreshes every few seconds and shows you what's currently running, sorted by CPU usage by default — so whatever is eating the most resources sits at the top.

```bash
top
```

Useful keys while inside top:

| Key | What it does |
|---|---|
| `k` | Kill a process — it'll ask you for the PID |
| `M` | Sort by memory usage instead of CPU |
| `P` | Sort by CPU usage (default) |
| `q` | Quit |

If your computer suddenly feels slow and you don't know why, open `top` immediately. The greedy process will be right at the top of the list.

---

## Running processes in the background

By default, when you run a command, it takes over your terminal until it's done. You can't type anything else while it runs. If you're starting something like a browser or a long-running tool and you want your terminal back, add `&` to the end of the command.

```bash
firefox &
```

Output:
```
[1] 2341
```

That `[1]` is the job number. The `2341` is the PID. Firefox is now running in the background and your terminal is free to use.

---

## Moving processes between foreground and background

Once something is running, you can move it around.

**Suspend a foreground process** (pause it without killing it):
```bash
Ctrl + Z
```

**Send a suspended process to the background** (keep it running, just out of sight):
```bash
bg
```

**Bring a background process back to the foreground:**
```bash
fg
```

If you have multiple background jobs running, `fg` brings back the most recent one. To bring back a specific one, use its job number:

```bash
fg 2
```

See all your background jobs and their numbers:

```bash
jobs
```

---

## Killing processes

When you need to stop something, you use `kill` followed by the PID.

```bash
kill 2341
```

This sends a polite termination signal — it asks the process to shut itself down cleanly. Most of the time this works fine.

If the process is frozen or refusing to stop, you force it:

```bash
kill -9 2341
```

The `-9` flag sends a **SIGKILL** signal, which the process cannot ignore, catch, or delay. It gets terminated immediately, no questions asked. Think of the regular `kill` as asking someone to leave, and `kill -9` as physically removing them.

**Be careful with root-owned processes.** If you kill a system process owned by root without knowing what it does, you can destabilize or crash the system. Always check the COMMAND column in `ps aux` before killing something you're not sure about.

---

## Finding a PID without ps

If you already know the name of the process, you don't need to scroll through `ps aux` output. Use `pgrep`:

```bash
pgrep firefox
```

Returns just the PID. Clean and fast.

Or use `pidof`:

```bash
pidof firefox
```

Does the same thing. Personal preference which one you use.

---

## Why this matters for hacking

When you land on a system, one of the first things you do is run `ps aux` and look at what's running. You're looking for:

- **Antivirus or endpoint protection** — if you're about to run an exploit or drop a file, you need to know what's watching. Finding the process and killing it (if you have the permissions) is a common step.
- **Interesting services** — databases, web servers, internal tools. These tell you what the machine is being used for and what might be worth targeting.
- **Other users' processes** — if multiple users are logged in, their processes show up here too. That's useful information.

`ps aux` is a recon tool as much as it is a management tool.

---

## Command reference

| Command | What it does |
|---|---|
| `ps aux` | Snapshot of every running process |
| `top` | Live view of processes sorted by resource use |
| `kill PID` | Politely ask a process to stop |
| `kill -9 PID` | Force stop a process immediately |
| `pgrep name` | Get the PID of a process by name |
| `pidof name` | Same as pgrep, alternate syntax |
| `jobs` | List all background jobs in current session |
| `fg` | Bring the last background job to the foreground |
| `bg` | Send a suspended job to the background |
| `command &` | Start a command in the background from the start |
| `Ctrl + Z` | Suspend a running foreground process |
| `updatedb` | (from last module) Just noting — needs root |

---

## A note on signals

`kill` doesn't just kill things — it sends **signals** to processes. `-9` (SIGKILL) is the most aggressive one. But there are others worth knowing eventually:

| Signal | Number | What it does |
|---|---|---|
| SIGTERM | 15 | Polite shutdown request (default) |
| SIGKILL | 9 | Immediate forced termination |
| SIGHUP | 1 | Reload config without restarting |

You'll mostly use 9 and 15. The others come up later when you're dealing with services and daemons.

---

## Practice

- Run `ps aux` and find your terminal process — note its PID
- Open `top` and watch it update. Find which process is using the most CPU right now
- Run `leafpad &` (or any text editor) in the background, then use `pgrep` to find its PID, then kill it with `kill -9`
- Try `Ctrl + Z` on a running command, then bring it back with `fg`

The kill-a-background-process flow is the one to get comfortable with. Run something, find its PID, kill it. Do that a few times until it feels automatic.

---

*Up next: Module 7 — Managing User Environment Variables*
