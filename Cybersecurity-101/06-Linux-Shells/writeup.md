# Linux Shells

## Overview

Linux Shells introduces the shell environment in Linux — what it is, the different types available, and how to write and execute basic shell scripts. The room builds on Linux Fundamentals by moving from individual commands into scripting, giving commands structure, logic, and automation capability.

---

## What is a Shell?

A shell is the interface between the user and the operating system. In Linux, the shell interprets commands typed by the user and passes them to the kernel for execution.

### Common Linux Shells

| Shell | Full Name | Notes |
|-------|-----------|-------|
| `bash` | Bourne Again Shell | Most common — default on most Linux distributions |
| `zsh` | Z Shell | Combines features of modern and older shells |
| `fish` | Friendly Interactive Shell | User-friendly with modern features |

---

## Commands Covered

| Command | Description |
|---------|-------------|
| `pwd` | Print working directory — shows current location |
| `grep` | Searches for a word or pattern inside a file |
| `chmod` | Changes file permissions — read, write, execute |

---

## Shell Scripting

### File Extension
Shell scripts use the `.sh` extension — for example: `first_script.sh`

### Shebang
The shebang is the first line of any shell script. It tells the system which interpreter to use when executing the script:

```bash
#!/bin/bash
```

The `#!` characters followed by the interpreter path must appear at the very top of the script file.

### Script Components Covered

- **Variables** — storing and referencing values
- **Loops** — repeating actions
- **Conditional statements** — logic and decision making — note that spacing inside `[ ]` is strict and must be followed exactly
- **Comments** — annotating code for readability

---

## File Permissions & chmod

When a new script is created, it does not automatically have execute permission. Attempting to run it without setting permissions will result in a **Permission denied** error.

### Permission Types

| Symbol | Permission |
|--------|------------|
| `r` | Read |
| `w` | Write |
| `x` | Execute |

### Granting Execute Permission

```bash
chmod +x script_name.sh
```

> **Personal Note:** This was not something TryHackMe's steps made obvious — the "Permission denied" error appeared when following the room instructions, and the fix required a separate Google search. It is worth remembering: **always `chmod` before running a new script.**

---

## Creating & Running a Script

```bash
# Create and edit the script
nano first_script.sh

# Grant execute permission
chmod +x first_script.sh

# Run the script
./first_script.sh
```
