# Linux Basics for Hackers
## Module 17 - Python Scripting

---

## Overview

Bash scripting gets you far for simple automation and quick one-liners. Python is what you reach for when things get more complex - when you need to parse data, work with networks, build tools that make decisions, or understand the exploit code that other people have written. This module covers the Python fundamentals you need as a foundation for security work.

---

## Why Python for security work

Python has been the dominant language in security and hacking for a long time. The reasons aren't complicated:

- The syntax is clean and readable - you can understand what a script does quickly
- The standard library covers networking, file I/O, cryptography, and more without installing anything extra
- Thousands of third-party libraries exist for security-specific tasks
- Most exploits, tools, and proof-of-concept code you'll find online are written in Python
- It runs everywhere Linux does

Being able to read Python is genuinely useful even before you're writing your own tools. Most exploits you'll encounter are Python scripts. If you can't read them, you're working blind.

---

## Python 2 vs Python 3

Python 2 is dead - officially end-of-life since 2020. Everything you write should be Python 3. Kali ships with Python 3 as the default. When you see old tutorials using `print "hello"` without parentheses, that's Python 2 syntax. In Python 3 it's `print("hello")`.

Check your version:

```bash
ahegazy0@kali:~$ python3 --version
```

Run the Python 3 interpreter:

```bash
ahegazy0@kali:~$ python3
```

This drops you into an interactive shell where you can type Python directly. Good for testing small things. Exit with `exit()` or `Ctrl+D`.

---

## Your first script

Create a file called `hello.py`:

```python
#!/usr/bin/env python3

print("Hello, world")
```

The shebang line is slightly different from Bash - `/usr/bin/env python3` finds the Python 3 interpreter wherever it lives on the system, which is more portable than hardcoding the path.

Run it:

```bash
ahegazy0@kali:~$ python3 hello.py
```

Or make it executable and run it directly:

```bash
ahegazy0@kali:~$ chmod 755 hello.py
ahegazy0@kali:~$ ./hello.py
```

---

## Variables and data types

Python is dynamically typed - you don't declare types, you just assign values and Python figures it out.

```python
name = "Alice"           # string
port = 80                # integer
pi = 3.14                # float
active = True            # boolean
```

**Strings** - text, always in quotes:

```python
target = "192.168.1.1"
print("Scanning: " + target)
print(f"Scanning: {target}")    # f-string, cleaner way to embed variables
```

**Integers** - whole numbers, used for ports, counts, indexes:

```python
port = 22
print(port + 1)    # 23
```

**Lists** - ordered collections, like arrays:

```python
ports = [22, 80, 443, 3306]
print(ports[0])      # 22 - indexing starts at 0
print(ports[-1])     # 3306 - negative index counts from the end
```

**Dictionaries** - key-value pairs, like a lookup table:

```python
user = {"username": "admin", "password": "password123", "role": "root"}
print(user["username"])    # admin
```

Dictionaries are very common in security scripts - you'll see them used for storing parsed data, HTTP headers, configuration values, and more.

---

## Getting input from the user

```python
target = input("Enter target IP: ")
print("Scanning " + target)
```

`input()` pauses and waits, then stores whatever the user types as a string. If you need it as a number:

```python
port = int(input("Enter port: "))
```

`int()` converts the string to an integer. Without this conversion, `"80" + 1` would throw an error - Python doesn't silently mix types.

---

## Conditionals

```python
password = input("Enter password: ")

if password == "secretpass":
    print("Access granted")
elif password == "admin":
    print("Admin access")
else:
    print("Wrong password")
```

Python uses indentation to define code blocks - no curly braces. The standard is 4 spaces. If your indentation is inconsistent, Python will throw an error. This is the thing that trips up almost every beginner coming from another language.

Common comparison operators:

| Operator | Meaning |
|---|---|
| `==` | Equal to |
| `!=` | Not equal to |
| `>` `<` | Greater / less than |
| `>=` `<=` | Greater or equal / less or equal |
| `in` | Checks if a value exists in a list or string |

---

## Loops

**For loop** - iterate over a list or range:

```python
ports = [22, 80, 443]
for port in ports:
    print(f"Checking port {port}")
```

```python
for i in range(1, 256):
    print(f"192.168.1.{i}")
```

`range(1, 256)` generates numbers from 1 to 255. This is how you'd build a basic IP range to scan.

**While loop** - keep going while a condition is true:

```python
attempts = 0
while attempts < 3:
    password = input("Password: ")
    if password == "secret":
        print("Access granted")
        break
    attempts += 1
print("Too many attempts")
```

`break` exits the loop immediately. `continue` skips to the next iteration.

---

## Functions

Functions let you write a block of code once and call it by name whenever you need it:

```python
def scan_port(ip, port):
    print(f"Scanning {ip}:{port}")

scan_port("192.168.1.1", 80)
scan_port("192.168.1.1", 443)
```

Functions can return values:

```python
def add(a, b):
    return a + b

result = add(3, 4)
print(result)    # 7
```

Well-structured scripts put logic in functions and call them from the bottom of the file. It makes the code readable and reusable.

---

## Importing libraries

Python's real power comes from its libraries. You bring them in with `import`:

```python
import socket
import os
import sys
```

**socket** - for network connections, the foundation of network tools:

```python
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("google.com", 80))
print("Connected")
s.close()
```

This opens a TCP connection to google.com on port 80 - the same thing a browser does when you visit a site over HTTP.

**os** - for interacting with the operating system:

```python
import os

os.system("ls -la")           # run a shell command
cwd = os.getcwd()             # get current directory
files = os.listdir(".")       # list files in current directory
```

**sys** - for system-level stuff like command-line arguments:

```python
import sys

print(sys.argv)               # list of arguments passed to the script
# python3 script.py 192.168.1.1 80
# sys.argv = ['script.py', '192.168.1.1', '80']
```

This lets you pass arguments to your script when you run it, rather than hardcoding values.

---

## Installing third-party libraries

The standard library covers a lot, but the Python ecosystem has hundreds of thousands of third-party packages for specialized tasks:

```bash
ahegazy0@kali:~$ pip3 install requests
ahegazy0@kali:~$ pip3 install scapy
ahegazy0@kali:~$ pip3 install paramiko
```

- `requests` - cleaner HTTP requests than using socket directly
- `scapy` - powerful packet crafting and analysis
- `paramiko` - SSH connections in Python

Import after installing:

```python
import requests

response = requests.get("http://example.com")
print(response.status_code)
print(response.text)
```

---

## A practical example - basic port scanner

This brings together most of what's above into something actually useful:

```python
#!/usr/bin/env python3
import socket
import sys

def scan_port(ip, port):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(1)
    result = s.connect_ex((ip, port))   # returns 0 if connection succeeded
    s.close()
    return result == 0

target = input("Enter target IP: ")
print(f"\nScanning {target}...\n")

for port in range(1, 1025):
    if scan_port(target, port):
        print(f"Port {port} is OPEN")

print("\nScan complete.")
```

`connect_ex` tries to connect and returns an error code rather than throwing an exception - 0 means success (port is open). `settimeout(1)` means don't wait more than 1 second per port before moving on.

This is a simplified version of what nmap does. Understanding it makes nmap's output make a lot more sense.

---

## Error handling

Things go wrong. Network connections fail, files don't exist, users type the wrong thing. Python uses `try/except` to handle this gracefully:

```python
try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(("192.168.1.1", 80))
    print("Connected")
except socket.error as e:
    print(f"Connection failed: {e}")
finally:
    s.close()
```

`try` - attempt this
`except` - if it fails, run this instead
`finally` - run this no matter what (cleanup code goes here)

Without error handling, one failed connection crashes the entire script. With it, your script keeps running and tells you what went wrong.

---

## Command Reference

| Command | What it does |
|---|---|
| `python3 script.py` | Run a Python script |
| `python3` | Open the interactive Python shell |
| `pip3 install [package]` | Install a third-party library |
| `pip3 list` | Show installed packages |
| `pip3 show [package]` | Show details about an installed package |

---

## Python cheat sheet

| Concept | Syntax |
|---|---|
| Print | `print("text")` |
| Variable | `name = "value"` |
| User input | `x = input("prompt: ")` |
| String formatting | `f"Hello {name}"` |
| If/else | `if x == y:` / `else:` |
| For loop | `for item in list:` |
| While loop | `while condition:` |
| Function | `def name(params):` |
| Import | `import socket` |
| List | `items = [1, 2, 3]` |
| Dictionary | `d = {"key": "value"}` |
| Try/except | `try:` / `except Error as e:` |

---

## Practice

- [ ] Write a script that asks for your name and prints a greeting using an f-string
- [ ] Write a script that loops through ports 20–25 and prints each one
- [ ] Build the port scanner above and test it against `127.0.0.1` (your own machine) - see which ports are open on yourself
- [ ] Write a script that asks for a password and prints "correct" or "incorrect" based on a hardcoded value
- [ ] Once you're comfortable with those: add error handling to the port scanner so a network error doesn't crash the whole scan

> 💡 *For deeper practice, I also recommend completing the end-of-chapter exercises in the official **Linux Basics for Hackers** book.*
---

## Where to go from here

This module is a foundation. The book ends here, but Python for security work goes much deeper:

- **Scapy** - craft and send custom packets, build your own scanners and sniffers at the packet level
- **Paramiko** - automate SSH connections, useful for scripting access to remote systems
- **Requests + BeautifulSoup** - web scraping and HTTP interaction
- **Subprocess** - call system commands from Python and capture their output
- Reading existing exploit code - most CVE proof-of-concepts on GitHub are Python. Being able to read and modify them is one of the most practical skills in this field.

The pattern going forward is the same as it's been throughout this course: understand the concept, run the commands, break things in your VM, and look things up when they don't work. That's how this stuff actually gets learned.

---

*End of Linux Basics for Hackers - 17 Modules Complete.*
