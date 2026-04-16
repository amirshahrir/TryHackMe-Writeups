# Windows PowerShell

## Overview

Windows PowerShell introduces cmdlets — PowerShell's command structure built around a strict **verb-noun convention**. Unlike CMD which uses short legacy commands, PowerShell commands are designed to be descriptive and consistent. The room covers navigation, file management, system information gathering, piping, filtering, and scripting — making it one of the most security-relevant rooms in the chapter.

---

## Understanding Cmdlets

PowerShell commands are called **cmdlets** (command-lets). They follow a `Verb-Noun` format where:
- The **verb** describes the action being performed
- The **noun** specifies the object the action targets

This makes commands readable and logical — but the learning curve comes from not always knowing the exact noun to pair with a verb. You know the concept but not necessarily the precise word. For example, knowing `Get-Something` exists is intuitive, but knowing whether it is `Get-NetIPAddress` or `Get-NetworkAddress` requires familiarity.

---

## Commands Covered

### Help & Discovery

| Cmdlet | Description |
|--------|-------------|
| `Get-Command` | Lists all available cmdlets |
| `Get-Command -CommandType "Function"` | Filters by command type |
| `Get-Command -Name Remove*` | Lists all commands starting with "Remove" |
| `Get-Help command_name` | Provides detailed information about a cmdlet |
| `Get-Help Get-Date -examples` | Shows usage examples for a cmdlet |
| `Get-Alias -Name echo` | Finds the cmdlet that has "echo" as its alias |

> **Personal Note:** Discovered `Get-Command -Name Remove*` by testing things out rather than being told — a good reminder that experimenting in PowerShell is one of the fastest ways to learn it.

### Navigation & File Management

| Cmdlet | Description |
|--------|-------------|
| `Set-Location` | Navigate to a different directory — equivalent to `cd` |
| `Get-ChildItem` | Lists files and directories — equivalent to `ls` in Linux or `dir` in CMD |
| `Get-ChildItem -Path "path"` | Lists contents of a specified path |
| `New-Item` | Creates a new file or directory — supports inline path and type |
| `Remove-Item` | Removes both files and directories |
| `Get-Content -Path "path"` | Displays the contents of a file |

### System Information

| Cmdlet | Description |
|--------|-------------|
| `Get-ComputerInfo` | Returns detailed information about the system |
| `Get-LocalUser` | Lists all local user accounts on the system |
| `Get-Process` | Displays all currently running processes in detail |
| `Get-Service` | Retrieves the status of services — running, stopped, or paused |

> **Note:** `Get-Service` is used extensively by system administrators and forensic analysts to quickly assess the state of a machine.

### Networking

| Cmdlet | Description |
|--------|-------------|
| `Get-NetIPConfiguration` | Detailed network info — IP address, DNS server, gateway |
| `Get-NetIPAddress` | Shows details for all IP addresses configured on the system |
| `Get-NetTCPConnection` | Displays current TCP connections — local and remote endpoints |

> **Most Useful for Security Work:** `Get-NetIPAddress` — when working in PowerShell, this is likely one of the most frequently used commands for quickly understanding the network configuration of a machine.

### Security & Forensics

| Cmdlet | Description |
|--------|-------------|
| `Get-FileHash` | Generates a file hash — used in incident response, threat hunting and malware analysis |

> Generating file hashes is something a cybersecurity professional does routinely — the same way a marketing professional sends a PowerPoint after completing their work. It is a standard part of the job, not an exceptional action.

### Piping & Filtering

| Operator / Cmdlet | Description |
|-------------------|-------------|
| `\|` | Pipes output of one command as input to the next |
| `Sort-Object` | Sorts output |
| `Where-Object` | Filters output based on conditions |
| `Select-Object` | Selects specific properties from objects |
| `Select-String` | Searches for text patterns within files |

**Comparison Operators used with `Where-Object`:**

| Operator | Meaning |
|----------|---------|
| `-eq` | Equal to |
| `-ne` | Not equal |
| `-gt` | Greater than |
| `-ge` | Greater than or equal to |
| `-lt` | Less than |
| `-le` | Less than or equal to |
| `-like` | Matches a pattern |

---

## Scripting

The room included hands-on scripting exercises where scripts were written and executed directly. PowerShell scripting is essentially giving the computer a structured to-do list.

### Blue Team Use Cases
- Log analysis
- Detecting anomalies
- Identifying Indicators of Compromise (IOCs)

### Red Team Use Cases
- Automating enumeration
- Executing remote commands
- Crafting obfuscated scripts to bypass defences

### Key Cmdlet for Remote Execution

```powershell
Invoke-Command -ScriptBlock { command_here }
```

Used to execute commands on remote systems — relevant for both legitimate administration and offensive security operations.
