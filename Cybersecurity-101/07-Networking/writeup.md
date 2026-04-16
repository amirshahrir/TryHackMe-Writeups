# Networking

## Overview

The Networking chapter covers both the theoretical foundations of how networks operate and the practical tools used to analyse, monitor and scan them. Theory rooms establish the concepts — OSI model, TCP/IP, core and secure protocols — while the hands-on rooms build the skills to interact with live and captured network traffic.

---

## Network Concepts — OSI & TCP/IP

> *📝 Notes coming soon*

---

## Network Essentials

> *📝 Notes coming soon*

---

## Networking Core Protocols — TCP/IP

> *📝 Notes coming soon*

---

## Networking Secure Protocols — TLS, SSH, VPN

> *📝 Notes coming soon*

---

## Wireshark

### What is Wireshark?

Wireshark is an open-source, cross-platform network packet analyser. It is one of the most widely used tools for inspecting network traffic — both live captures and pre-recorded packet files.

### Capabilities

- Sniffing and investigating live traffic
- Inspecting packet captures (PCAP files)
- Detecting and troubleshooting network problems such as load failures and congestion
- Detecting security anomalies — rogue hosts, abnormal port usage, suspicious traffic
- Investigating protocol details such as response codes and payload data

> **Note:** Wireshark is not an IDS (Intrusion Detection System). It discovers and investigates traffic but does not modify packets or generate alerts automatically.

### Working with PCAPs

The room used pre-made PCAP files rather than live capture. When working with a PCAP the key is not to get overwhelmed by the volume of data — focus on the **Pane** relevant to what you are investigating rather than trying to read every packet.

### Coloring Rules

Wireshark uses color coding to help analysts quickly identify different types of traffic at a glance.

- Access via **View → Coloring Rules**
- Each color maps to a specific protocol or condition as defined in the color guide
- Useful for visually separating normal traffic from anomalies during analysis

---

## TCPDump

### What is TCPDump?

TCPDump is a command-line packet analyser. Where Wireshark provides a GUI, TCPDump operates entirely in the terminal — making it useful in environments where a GUI is not available, such as remote servers or minimal Linux installs.

### Commands & Flags

#### Interface Selection

| Command | Description |
|---------|-------------|
| `tcpdump -i INTERFACE` | Capture on a specific interface |
| `tcpdump -i any` | Capture on all available interfaces |
| `tcpdump -i eth0` | Capture on the eth0 interface |
| `ip address show` / `ip a s` | List available network interfaces |

#### Capture Control

| Command | Description |
|---------|-------------|
| `-w FILE` | Save captured packets to a file for later inspection in Wireshark |
| `-r FILE` | Read packets from a saved file |
| `-c COUNT` | Specify the number of packets to capture then stop |

#### Output Control

| Command | Description |
|---------|-------------|
| `-n` | Avoid DNS lookup — shows raw IP addresses |
| `-nn` | Stops both DNS and port number lookups |
| `-v` | Verbose output — more detail per packet |
| `-vv` / `-vvv` | Increasing levels of verbosity |

#### Filtering

| Filter | Example | Description |
|--------|---------|-------------|
| Host | `tcpdump src host IP` | Filter by source IP address |
| Hostname | `tcpdump host HOSTNAME` | Filter by hostname |
| Port | `tcpdump port 53` | Filter by port number |
| Protocol | `tcpdump icmp` | Filter by protocol |

**Supported protocols for filtering:** `ip`, `ip6`, `udp`, `tcp`, `icmp`

**Example:**
```bash
sudo tcpdump -i ens5 icmp -n
```

---

## Nmap

### What is Nmap?

Nmap (Network Mapper) is an open-source tool for network discovery and security auditing. It is used to discover hosts on a network, scan for open ports, detect running services and their versions, and identify operating systems.

### Target Specification

| Method | Example | Description |
|--------|---------|-------------|
| IP range | `192.168.0.1-10` | Scans all IPs from .1 to .10 |
| Subnet | `192.168.0.1/24` | Scans entire /24 subnet |
| Hostname | `example.thm` | Scans by hostname |

### Host Discovery

The primary focus in this room was **checking if hosts are alive on a network** — identifying which machines are up before deeper scanning.

| Flag | Description |
|------|-------------|
| `-sn` | Ping scan — discovers live hosts without port scanning |
| `-sL` | List scan — calculates IPs in a range and looks up DNS names without sending packets. A stealthy way to map a range |
| `-v` | Verbose — shows what Nmap is doing in real time. Without this, the terminal appears blank during a scan |

**Example:**
```bash
nmap -sn -v 192.168.66.0/24
```

### Port Scanning

| Flag | Description |
|------|-------------|
| `-F` | Fast mode — scans the 100 most common ports instead of the default 1000 |
| `-p10-1024` | Scans ports 10 to 1024 |
| `-p-25` | Scans ports 1 to 25 |
| `-p-` | Scans all 65535 ports — most thorough option |
| `-p1-1023` | Scans well-known ports — the most common services live here |

> **Tip:** The most common services use ports between 1 and 1024, known as well-known ports.

### Scan Types

| Flag | Scan Type | Description |
|------|-----------|-------------|
| `-sT` | TCP Connect | Completes the full three-way handshake |
| `-sS` | TCP SYN | Only performs the first step of the handshake — stealthier |
| `-sU` | UDP Scan | Scans UDP ports |

### Detection Flags

| Flag | Description |
|------|-------------|
| `-O` | OS detection |
| `-sV` | Service and version detection |
| `-A` | OS detection, version detection and additional information combined |
| `-Pn` | Treat all hosts as online — scans hosts that appear to be down |

### Scanning Speed

| Flag | Name | Estimated Time |
|------|------|---------------|
| `-T0` | Paranoid | ~9.8 hours |
| `-T1` | Sneaky | ~27.53 minutes |
| `-T2` | Polite | ~40.56 seconds |
| `-T3` | Normal | ~0.15 seconds |
| `-T4` | Aggressive | ~0.13 seconds |

> Slower speeds like T0 and T1 are used in real engagements to evade detection — the slower the scan, the less likely it triggers IDS alerts.

### Fine-Tuning Performance

| Flag | Description |
|------|-------------|
| `--min-parallelism <n>` | Minimum number of parallel service probes |
| `--max-parallelism <n>` | Maximum number of parallel service probes |
| `--min-rate <n>` | Minimum packets per second |
| `--max-rate <n>` | Maximum packets per second |
| `--host-timeout <time>` | Maximum time to wait for a target host to respond |
