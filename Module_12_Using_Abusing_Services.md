# Linux Basics for Hackers
## Module 12 — Using & Abusing Services

---

## What this module is about

Services are programs that run in the background continuously, waiting to do something — serve a web page, accept a database query, answer an SSH connection. They're the working parts of a server. Understanding them matters both for setting up your own tools and for understanding what's running on a target machine and how to get in through it.

---

## What a service actually is

A service (also called a daemon) is a process that starts at boot and keeps running indefinitely in the background. It doesn't have a window or a visible interface. It just sits there listening for requests on a specific network port, and responds when something connects to it.

Examples:
- Apache listens on port 80 (HTTP) and 443 (HTTPS) — serves web pages
- SSH listens on port 22 — accepts remote terminal connections
- MySQL listens on port 3306 — answers database queries
- PostgreSQL listens on port 5432 — same idea, different database

The port number is how you know which service you're talking to when you connect to a remote machine. This becomes very important later when you're doing port scanning with nmap — each open port is a service, and each service is a potential way in.

---

## Starting and stopping services

There are two tools for managing services: `service` (the older way) and `systemctl` (the modern standard). Both work in Kali. `systemctl` is what you'll see most on current systems.

**Using service:**

```bash
service apache2 start
service apache2 stop
service apache2 restart
service apache2 status
```

**Using systemctl:**

```bash
systemctl start apache2
systemctl stop apache2
systemctl restart apache2
systemctl status apache2
```

They do the same thing. `systemctl` gives you more detailed status output and is the direction things have moved, so it's worth learning both but defaulting to `systemctl`.

**Enable a service to start automatically at boot:**

```bash
systemctl enable apache2
```

**Disable auto-start:**

```bash
systemctl disable apache2
```

---

## Apache — the web server

Apache is one of the most widely used web servers in the world. Starting it turns your machine into a web server.

```bash
service apache2 start
```

Once it's running, open a browser and go to `http://localhost`. You'll see the default Apache page. That page is just a file sitting at `/var/www/html/index.html`. Replace it with your own content and that's what gets served.

```bash
echo "<h1>My custom page</h1>" > /var/www/html/index.html
```

Refresh the browser — your content is now being served.

Apache logs everything: every request, every IP that connected, every file that was requested. Those logs are in `/var/log/apache2/`. The access log shows who connected and what they asked for. The error log shows what broke.

**Why this matters for hacking:** Apache has had many vulnerabilities over the years. An unpatched or misconfigured Apache server is a common entry point. When you scan a target and see port 80 or 443 open, Apache (or nginx, or another web server) is almost certainly behind it. The version number matters — older versions have known exploits.

---

## SSH — remote terminal access

SSH (Secure Shell) lets you control another computer's terminal over the network, encrypted. It's the standard way to administer remote Linux systems.

```bash
ssh username@192.168.1.50
```

This opens an interactive terminal session on the machine at that IP, logged in as `username`. Everything you type is encrypted in transit.

**Starting the SSH service on your machine** (so others can connect to you):

```bash
service ssh start
```

**Connecting with a specific port** (if the server isn't on the default port 22):

```bash
ssh -p 2222 username@192.168.1.50
```

**Copying files over SSH with scp:**

```bash
scp file.txt username@192.168.1.50:/home/username/
```

This copies `file.txt` to the remote machine. Works in both directions — you can pull files from a remote machine the same way.

**Why this matters for hacking:** SSH is everywhere. If you get valid credentials on a machine with SSH running, you have full remote access to it. Weak passwords on SSH accounts are one of the most common ways servers get compromised. Tools like Hydra can brute-force SSH login if the server doesn't have rate limiting or fail2ban configured.

Kali ships with a default password on the SSH service. Change it immediately if you enable it. Running Kali with default credentials and SSH open is a serious mistake.

---

## MySQL — the database

MySQL is a relational database server. It stores data in tables and answers queries written in SQL. It's behind the majority of web applications — login systems, user data, product listings, everything.

**Starting MySQL:**

```bash
service mysql start
```

**Logging into the MySQL shell:**

```bash
mysql -u root -p
```

`-u root` means log in as the root database user. `-p` means prompt for a password. Once you're in, you get a MySQL prompt:

```sql
mysql>
```

Basic SQL to know from here:

```sql
SHOW DATABASES;           -- list all databases
USE database_name;        -- switch to a database
SHOW TABLES;              -- list tables in the current database
SELECT * FROM users;      -- dump everything from the users table
```

**Why this matters for hacking:** SQL injection is one of the most common web vulnerabilities. If a web application doesn't properly sanitize input, you can inject SQL commands through a login form or URL and extract data from the database directly. Even without SQL injection, finding a MySQL server with a weak or default root password gives you access to everything stored in it — usernames, password hashes, email addresses, private data.

---

## PostgreSQL

PostgreSQL is another database server, similar to MySQL. The reason it's worth mentioning here specifically is that **Metasploit** — the main exploitation framework you'll use later in this course — uses PostgreSQL as its backend to store scan results and session data.

```bash
service postgresql start
```

You won't interact with it directly much. Just know that when you start Metasploit, it'll tell you the database isn't connected if PostgreSQL isn't running first.

---

## Checking what services are running

To see all services currently active on your system:

```bash
systemctl list-units --type=service --state=running
```

To see which ports are currently open and what's listening on them:

```bash
ss -tlnp
```

Or the older equivalent:

```bash
netstat -tlnp
```

Output shows you the port number, the protocol, and which process is listening. This is useful both for your own machine (knowing what you're exposing) and on a target (if you have access to the machine and want to see what services are running internally).

---

## Default credentials — a real problem

Kali Linux ships with default passwords for several services. So do many routers, cameras, databases, and servers by default. "Admin/admin", "root/root", "admin/password" — these are credentials that never got changed after installation.

A huge portion of real-world breaches happen this way. Not through clever exploits or zero-days — through someone just trying the default username and password and getting in.

When you set up any service, change the default credentials immediately. When you're assessing a target, trying default credentials is always one of the first steps.

---

## Command reference

| Command | What it does |
|---|---|
| `service name start/stop/restart` | Manage a service (older syntax) |
| `systemctl start/stop/restart name` | Manage a service (modern syntax) |
| `systemctl status name` | Check if a service is running |
| `systemctl enable name` | Start service automatically at boot |
| `systemctl disable name` | Disable auto-start |
| `systemctl list-units --type=service` | List all running services |
| `ssh user@ip` | Connect to a remote machine via SSH |
| `scp file user@ip:/path` | Copy a file to a remote machine |
| `mysql -u root -p` | Log into MySQL shell |
| `ss -tlnp` | Show open ports and listening services |

---

## Key services and their ports

| Service | Default port | What it does |
|---|---|---|
| SSH | 22 | Remote terminal access |
| HTTP (Apache) | 80 | Web server |
| HTTPS | 443 | Web server (encrypted) |
| MySQL | 3306 | Database |
| PostgreSQL | 5432 | Database |
| FTP | 21 | File transfer |
| SMTP | 25 | Email sending |

Memorizing these is worth doing. When nmap returns a list of open ports, knowing what's normally on each port tells you immediately what services you're looking at.

---

## Practice

- Start Apache with `service apache2 start`, then open a browser and go to `http://localhost`
- Replace the content of `/var/www/html/index.html` with something custom and refresh the browser
- Start MySQL and log in with `mysql -u root -p`, then run `SHOW DATABASES;`
- Run `ss -tlnp` and see which ports are open after starting those services
- Stop both services when you're done — don't leave things running you don't need

---

*Up next: Module 13 — Becoming Secure & Anonymous*
