# Linux Basics for Hackers
## Module 7 - Environment Variables

---

## Overview

Your Linux system runs on a set of behind-the-scenes settings that control how everything behaves - where it looks for programs, who you are, what your home folder is, what your terminal prompt looks like. These are called **environment variables**. They're always there, you just haven't looked at them yet.

---

## What environment variables actually are

An environment variable is just a named value the system keeps in memory while you're logged in. Programs and the shell itself read these values constantly to decide how to behave.

Some are set by the system at boot. Some are set when you log in. And you can create or change your own at any time.

The naming convention is simple: variable names are usually all caps, and you reference them by putting a `$` in front of the name.

```bash
ahegazy0@kali:~$ echo $HOME
/home/kali
```

---

## The important variables to know

| Variable | What it holds |
|---|---|
| `$PATH` | List of folders the system searches when you type a command |
| `$HOME` | Your home folder path |
| `$USER` | Your current username |
| `$SHELL` | Which shell you're running (usually /bin/bash) |
| `$HISTSIZE` | How many commands your history file remembers |
| `$PS1` | What your command prompt looks like |

---

## PATH - the most important one

`$PATH` is a list of folder paths separated by colons. When you type a command like `ls`, the system doesn't magically know where `ls` lives. It goes through every folder listed in `$PATH`, one by one, until it finds a program with that name.

```bash
ahegazy0@kali:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Each folder separated by `:` is a place the system will look.

This is why you get "command not found" sometimes. You installed a tool, but its folder isn't in your PATH, so the system has no idea where to look for it. The fix is adding that folder to PATH, which is covered below.

**If you break your PATH variable**, you lose the ability to run almost any command because the system can't find them anymore. This is fixable but annoying. Be careful when editing it.

---

## Viewing all your environment variables

```bash
ahegazy0@kali:~$ env
```

This prints every environment variable currently set in your session. There'll be more than you expect. Scroll through it and you'll recognize most of them.

---

## Changing and creating variables

To create or change a variable just for your current terminal session:

```bash
ahegazy0@kali:~$ MYVAR="hello"
ahegazy0@kali:~$ echo $MYVAR
hello
```

The problem is this disappears the moment you close the terminal. To make it available to any child processes (programs you run from your terminal), you need to **export** it:

```bash
ahegazy0@kali:~$ export MYVAR="hello"
```

Now any program launched from that terminal can read `$MYVAR`.

To add a folder to your PATH without overwriting the existing one:

```bash
ahegazy0@kali:~$ export PATH=$PATH:/new/folder/here
```

The `$PATH` at the start keeps everything that was already there. You're just appending `:new/folder/here` to the end. If you forget that part and just write `/new/folder/here`, you wipe everything else and break your PATH entirely.

---

## Making changes permanent

`export` only lasts for the current session. To make a variable stick across reboots and new terminals, you add it to your shell's config file.

For bash, that file is `~/.bashrc`. Open it and add your export line at the bottom:

```bash
ahegazy0@kali:~$ export PATH=$PATH:/your/new/tool/folder
```

Save the file. Then either open a new terminal or run:

```bash
ahegazy0@kali:~$ source ~/.bashrc
```

That reloads the file without needing to restart.

---

## Changing your prompt - PS1

`$PS1` controls what your command prompt looks like. By default it shows something like `kali@kali:~$`. You can change it to anything.

```bash
ahegazy0@kali:~$ export PS1="Hacker-Level-99: # "
```

Your prompt now looks like:
```
Hacker-Level-99: # 
```

It's mostly cosmetic, but some people set their prompt to show useful info - the current directory, the git branch they're on, the time. For now just know that PS1 is what controls it and you can edit it.

---

## The history trick hackers use

`$HISTSIZE` controls how many commands get saved to your history file. When you press the up arrow and scroll through past commands, that's history.

```bash
ahegazy0@kali:~$ export HISTSIZE=0
```

Setting it to 0 means nothing gets saved. The terminal stops logging your commands for the rest of that session. After you close the terminal, anyone who opens the history file won't see what you ran.

This is a basic operational security move. If you're on a system you're not supposed to be on and you don't want to leave traces, this is one of the first things you do.

---

## Command Reference

| Command | What it does |
|---|---|
| `env` | Print all current environment variables |
| `echo $VAR` | Print the value of a specific variable |
| `MYVAR="value"` | Create a variable for the current session |
| `export MYVAR="value"` | Create a variable available to child processes too |
| `export PATH=$PATH:/folder` | Add a folder to PATH without breaking it |
| `source ~/.bashrc` | Reload your bash config file without restarting |

---

## Practice

- [ ] Run `env` and find your `$SHELL` variable - it'll tell you exactly what shell you're running
- [ ] Run `echo $PATH` and look at every folder listed - those are all the places your system looks for programs
- [ ] Try `export HISTSIZE=0` then press the up arrow - your history is gone for this session
- [ ] Change your PS1 to display just your name, or something custom, and watch the prompt update immediately

> 💡 *For deeper practice, I also recommend completing the end-of-chapter exercises in the official **Linux Basics for Hackers** book.*
---

*Up next: Module 8 - Bash Scripting*
