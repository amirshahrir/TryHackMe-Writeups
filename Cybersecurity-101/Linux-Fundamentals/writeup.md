# Linux Fundamentals (Parts 1, 2 & 3) — TryHackMe

**Path:** Cyber Security 101  

---

## Overview

Linux Fundamentals is a three-part series that builds from basic terminal usage all the way to system administration concepts relevant to real security work.

- **Part 1** covers the essential commands needed to navigate and interact with a Linux terminal from scratch
- **Part 2** introduces SSH for remote machine access and expands into deeper filesystem interaction and navigation
- **Part 3** goes further into terminal text editors, general utilities, automation with cron, package management, and reading system logs

---

## Commands Used

| Command | What It Does |
|---------|-------------|
| `ls` | List files and directories in current location |
| `cd` | Change directory |
| `cd ..` | Move up one directory level |
| `cat` | Read and display file contents |
| `find` | Search for files across the filesystem |
| `touch` | Create a new empty file |
| `mkdir` | Create a new directory |
| `cp` | Copy a file |
| `mv` | Move or rename a file |
| `rm` | Remove/delete a file |
| `file` | Identify what type of file something is |
| `su` | Switch user account |
| `wget` | Download files from the internet |
| `ssh username@IPaddress` | Remotely log into a Linux machine |
| `crontab -e` | Edit the cron automation schedule |
| `crontab -l` | List current cron jobs |
| `rwx` | Read, write, execute — file permission flags |

---

## Key Concepts

### SSH — Remote Access
Part 2 introduced logging into a remote Linux machine using SSH. This is fundamental to cybersecurity work — almost every TryHackMe room and real-world engagement involves connecting to a remote machine this way.

```bash
ssh username@IPaddress
```

Managing multiple SSH terminal sessions simultaneously was one of the more practical skills in this section — logging into different machines across multiple terminal windows is a workflow that directly mirrors real penetration testing and SOC work.

### Cron — Automation
The standout concept from Part 3 was cron and crontab. Cron is Linux's built-in task scheduler — it runs commands automatically at defined times without any manual input.

```bash
crontab -e    # edit your scheduled tasks
crontab -l    # list what's currently scheduled
```

From a security perspective this is significant in two directions — defenders need to know what's scheduled on a system to detect malicious persistence, and attackers use cron jobs as a technique to maintain access after initial compromise.

### File Permissions (rwx)
Linux controls who can access files through a three-part permission system — read (r), write (w), and execute (x) — applied separately for the owner, group, and everyone else.

```bash
-rwxr-xr-- 
```
This means the owner can read, write and execute. The group can read and execute. Everyone else can only read.

---

## Challenges & How I Solved Them

The log section in Part 3 was the most challenging — there were no step-by-step instructions, only a question. This required applying everything learned across all three parts independently: changing directories, using `ls` and `cat` to navigate and read files, and reasoning about where log files are typically stored on a Linux system.

This is actually a realistic simulation of real security work — in an actual engagement nobody hands you instructions. You navigate, enumerate and figure it out.

---

## Why Linux Matters in Cybersecurity

Linux is not just another operating system — it is the dominant environment in cybersecurity. Most servers run Linux. Most attack tools (Nmap, Metasploit, Burp Suite) run on Linux. Most CTF target machines run Linux. Being fast and comfortable in the Linux terminal is a prerequisite for almost everything that comes later in this learning path.

The command line is faster than any GUI once you know what you are doing — and in security work, speed and precision in the terminal directly translates to effectiveness.

---

## What I Would Google to Go Deeper

- Essential Linux commands for penetration testing
- Linux commands used in cybersecurity and SOC work
- How cron jobs are used in persistence attacks
- Linux log file locations and what each log records

---

## Tools Used

- Linux terminal (TryHackMe AttackBox)
- SSH
- Crontab

---

*Part of my TryHackMe cybersecurity learning journey — transitioning from Frontend Development into Cybersecurity. CompTIA Security+ certified.*
