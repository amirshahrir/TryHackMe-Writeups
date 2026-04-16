# Windows Command Line

## Overview

Windows Command Line introduces `cmd.exe` — the native Windows command-line interpreter. The room covers four core areas: basic system information, network configuration and troubleshooting, file and directory management, and task and process management. It is the foundation for understanding how Windows can be interacted with outside of the GUI.

---

## What I Learned

The room is structured around four practical goals:

- Displaying basic system information
- Checking and troubleshooting network configuration
- Managing files and folders
- Checking and managing running processes

---

## Commands Covered

### Basic System Information

| Command | Description |
|---------|-------------|
| `set` | Displays environment variables including the system `Path` |
| `ver` | Displays the current Windows OS version |
| `systeminfo` | Shows detailed system information |
| `driverquery` | Lists all installed device drivers |
| `driverquery \| more` | Paginates the output — navigate with Spacebar |
| `help` | Provides help information for a specific command |
| `cls` | Clears the command prompt screen |

### Networking

| Command | Description |
|---------|-------------|
| `ipconfig` | Shows IP address, subnet mask and default gateway |
| `ipconfig /all` | Extended view including DNS servers and DHCP |
| `ping target_name` | Tests connectivity to a host — e.g. `ping example.com` |
| `tracert target_name` | Traces the network route taken to reach the target |
| `nslookup example` | Looks up a hostname or domain and returns its IP address |
| `netstat -abon` | Displays current network connections and listening ports |

> **Personal Note — TTL:** Learned that TTL (Time To Live) is a failsafe that prevents network packets from looping indefinitely. Each hop decrements the TTL value until it reaches zero and the packet is dropped.

### Directories

| Command | Description |
|---------|-------------|
| `cd` | Changes directory — without parameters, shows current location |
| `cd ..` | Goes up one level |
| `dir` | Lists contents of current directory |
| `dir /a` | Includes hidden and system files |
| `dir /s` | Lists files in current directory and all subdirectories |
| `dir /w` | Displays in wide column format |
| `tree` | Visually represents the directory and subdirectory structure |
| `mkdir directory_name` | Creates a new directory |
| `rmdir directory_name` | Removes a directory |

### Files

| Command | Description |
|---------|-------------|
| `type` | Displays the contents of a text file |
| `echo > filename.txt` | Creates a new text file in the current directory |
| `echo filename.txt` | Prints the filename to the screen |
| `copy file.txt renamed.txt` | Creates a copy of a file with a new name |
| `copy file.txt C:\path\` | Copies a file to a different directory |
| `more` | Displays output one page at a time — navigate with Spacebar or Enter |
| `del` / `erase` | Deletes a file — e.g. `erase text.txt` |

### Task & Process Management

| Command | Description |
|---------|-------------|
| `tasklist` | Lists all currently running processes |
| `tasklist /?` | Shows the help page with additional filter options |
| `tasklist /FI "imagename eq sshd.exe"` | Filters processes by image name |
| `taskkill /PID 4567` | Terminates a process by its Process ID |

> **PID (Process ID):** Each running process is assigned a unique ID. This ID can be used to target and terminate specific processes.

### Additional Commands

| Command | Description |
|---------|-------------|
| `chkdsk` | Checks the file system and disk for errors and bad sectors |
| `sfc /scannow` | Scans system files for corruption and repairs them if possible |
