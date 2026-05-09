# Linux Basics for Hackers
## Module 8 — Bash Scripting

---

## What this module is about

A hacker who types the same commands over and over is wasting time. Scripting is how you automate that — you write the commands once in a file, and then you just run the file. This module covers how to write Bash scripts, from a basic "hello world" to something that actually takes input and does something useful with it.

---

## What a script is

A script is a plain text file containing a list of commands. When you run it, Bash reads each line and executes it in order, the same as if you'd typed each command yourself. The difference is it happens in seconds and you never have to type it again.

You can put any command in a script that you'd run in the terminal. `ls`, `ping`, `nmap`, `grep` — all of it works. And you can combine them with logic: if this output contains this string, run this other command. That's where the power comes from.

---

## The shebang line

Every Bash script starts with this on the very first line:

```bash
#!/bin/bash
```

This is called the **shebang** (or hashbang). It tells the system exactly which program to use to interpret the file. Without it, Linux might not know what to do with the file — or it might use the wrong shell and your script breaks in confusing ways.

Always put it first. No blank line before it, nothing.

---

## Creating your first script

Open a text editor and create a file called `myscript.sh`:

```bash
#!/bin/bash
echo "Hello, world"
```

Save it. Now before you can run it, you need to make it executable:

```bash
chmod 755 myscript.sh
```

This gives you permission to run it as a program. Without this step, Linux will refuse to execute it and just tell you "Permission denied."

Now run it:

```bash
./myscript.sh
```

Output:
```
Hello, world
```

The `./` at the start means "run this file from the current directory." You need it because the current directory usually isn't in your PATH, so Linux won't find the script otherwise.

---

## echo — printing to the screen

`echo` prints whatever you give it to the terminal. It's how your script communicates back to you.

```bash
echo "Scan starting..."
echo "Done."
```

You can also echo variable values:

```bash
echo "Your username is: $USER"
```

Output:
```
Your username is: kali
```

---

## Variables in scripts

Variables let you store data and reuse it. You assign them like this — no spaces around the `=`:

```bash
name="Bob"
echo "Hello, $name"
```

Output:
```
Hello, Bob
```

When you want to use the value stored in a variable, put `$` in front of the name. When you're assigning to it, no `$`.

---

## read — taking input from the user

`read` pauses the script and waits for the user to type something, then stores what they typed in a variable.

```bash
#!/bin/bash
echo "What is your name?"
read name
echo "Hello, $name"
```

Run it and it waits. You type "Alice" and press enter:

```
What is your name?
Alice
Hello, Alice
```

You can also do it on one line with the `-p` flag (prompt):

```bash
read -p "Enter your name: " name
```

Cleaner. Does the same thing.

---

## A practical example — ping scanner

This is a simple script that asks for an IP address and pings it. It's the kind of thing you'd actually use:

```bash
#!/bin/bash
read -p "Enter IP address to ping: " target
echo "Pinging $target..."
ping -c 4 $target
```

The `-c 4` tells ping to send exactly 4 packets and stop. Without it, ping runs forever.

Save it as `pinger.sh`, `chmod 755 pinger.sh`, and run it with `./pinger.sh`.

---

## Comments

Any line starting with `#` is a comment. Bash ignores it completely. Use them to explain what your script is doing, especially for anything non-obvious.

```bash
#!/bin/bash
# This script pings a target IP address
# Written for practice - Module 8

read -p "Enter target IP: " target
ping -c 4 $target   # send 4 packets only
```

Comments are for you (and anyone else reading the script later). Write them as if you'll forget what this does in two weeks, because you will.

---

## Conditional logic — if/else

Once you can make decisions in a script, it becomes genuinely useful. Basic syntax:

```bash
#!/bin/bash
read -p "Enter a number: " num

if [ $num -gt 10 ]; then
    echo "That number is greater than 10"
else
    echo "That number is 10 or less"
fi
```

`fi` closes the `if` block (it's just `if` backwards). The `-gt` means "greater than". Common comparison operators:

| Operator | Meaning |
|---|---|
| `-eq` | Equal to |
| `-ne` | Not equal to |
| `-gt` | Greater than |
| `-lt` | Less than |
| `-ge` | Greater than or equal |
| `-le` | Less than or equal |

For comparing strings, use `=` and `!=` inside the brackets.

---

## Loops — doing something repeatedly

A `for` loop runs a block of commands multiple times:

```bash
#!/bin/bash
for i in 1 2 3 4 5; do
    echo "Pinging 192.168.1.$i"
    ping -c 1 192.168.1.$i
done
```

This pings five different IP addresses one after another. This is basically a primitive network scanner. Real scanners like nmap do this thousands of times with much more logic around it, but the concept is the same.

---

## Making scripts actually useful — the structure to follow

A script that's worth keeping usually has this shape:

```bash
#!/bin/bash
# Script name and what it does
# Your name, date

# --- Variables ---
target=""

# --- Input ---
read -p "Enter target: " target

# --- Main logic ---
echo "Running scan on $target..."
nmap -sV $target

# --- Done ---
echo "Scan complete."
```

Clean sections, comments on anything that needs explaining, and the logic flows top to bottom. You don't need to follow this exactly, but having some structure stops scripts from becoming unreadable after you write more than 20 lines.

---

## File permissions recap — chmod

When you create a script, it's a regular text file by default. To run it, you need the execute permission set.

```bash
chmod 755 myscript.sh
```

What `755` means:
- `7` — owner can read, write, and execute
- `5` — group can read and execute
- `5` — everyone else can read and execute

For personal scripts that only you need to run, `chmod 700` is fine — only you can do anything with it.

---

## Command reference

| Command / Concept | What it does |
|---|---|
| `#!/bin/bash` | Shebang — must be the first line of every script |
| `echo "text"` | Print text to the screen |
| `read var` | Take user input and store it in a variable |
| `read -p "prompt" var` | Same but with an inline prompt message |
| `name="value"` | Assign a value to a variable |
| `$name` | Use the value of a variable |
| `chmod 755 file.sh` | Make a script executable |
| `./script.sh` | Run a script in the current directory |
| `# comment` | A line Bash ignores — notes for humans |
| `if [ ] then / fi` | Conditional logic |
| `for x in ... do / done` | Loop over a list |

---

## Practice

- Write a script that asks for your name and prints "Hello, [name]"
- Make it executable with `chmod 755` and run it with `./`
- Write a second script that asks for an IP address and runs `ping -c 4` on it
- Once that works, modify it to loop through a range like `192.168.1.1` to `192.168.1.5` and ping each one

The ping loop exercise is worth doing properly. It's a real technique and understanding how it works makes nmap output make more sense later.

---

*Up next: Module 9 — Archiving & Compression*
