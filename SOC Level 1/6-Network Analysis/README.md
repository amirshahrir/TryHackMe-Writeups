# Network Traffic Analysis

---

## Overview

Network Traffic Analysis (NTA) is the chapter where SOC Level 1 moves from knowing what tools exist (EDR, Splunk, ELK) into actually reading the traffic those tools are supposed to be watching. It covers four TryHackMe rooms in order:

1. Network Traffic Basics
2. Wireshark: The Basics
3. Wireshark: Packet Operations
4. Wireshark: Traffic Analysis

The thread that runs through all four rooms is the same idea. Logs tell you that something happened. Traffic analysis tells you what actually happened. A firewall log can say a host made a DNS query. It cannot tell you that the DNS response carried a base64 encoded command back to a compromised machine. That gap between "logged" and "actually visible" is the reason NTA exists as its own discipline, and it is the reason a SOC analyst needs Wireshark skills on top of log review.

---

## 1. Network Traffic Basics

### What NTA Actually Is

NTA is the process of capturing, inspecting, and analyzing data as it moves across a network, with the goal of getting complete visibility into what is communicating with what, both inside and outside the network boundary.

It is worth being precise here because it is easy to conflate NTA with Wireshark. Wireshark is one tool used to do NTA. NTA itself is the broader discipline, made up of correlating multiple logs, deep packet inspection (DPI), and network flow statistics, all aimed at a specific investigative goal. Wireshark is how you look at packets. NTA is the practice of deciding what to look for and why.

### Why This Matters: The DNS Tunneling Example

The room uses a scenario that makes the log limitation problem concrete. A SOC analyst gets an alert for an unusual number of DNS queries from a host (WIN-016, 192.168.1.16). The firewall's DNS logs show several queries to the same top level domain, each using a different subdomain, for example:

```
2025-10-03 09:15:23 SRC=192.168.1.16 QUERY=aj39skdm.malicious-tld.com QTYPE=A
2025-10-03 09:15:45 SRC=192.168.1.16 QUERY=cmd01.malicious-tld.com QTYPE=TXT
```

From the log alone, an analyst can pull the query, the query type, the subdomain and TLD (checkable against AbuseIPDB or VirusTotal), the source host, and a timestamp. What the log cannot show is the content of the DNS reply. This matters because TXT records can be used to smuggle actual Command and Control (C2) instructions back to a compromised host. A firewall log confirms a query happened. Only the packet capture confirms what came back.

This is the core argument for NTA as a discipline separate from log review: standard logs record fragments of headers, never full packet content, and the fragments they do record are not enough to draw a solid conclusion on their own.

### Traffic Visibility Through the TCP/IP Stack

The room uses the TCP/IP stack as a framework for explaining exactly what gets logged versus what only shows up in full traffic capture, layer by layer.

**Application Layer.** Logs like web proxy or firewall logs typically capture header details (an HTTP GET request line, a response code) but exclude the payload. A logged `GET /downloads/suspicious_package.zip HTTP/1.1` request confirms a file was requested. The log will not show you the actual 10MB of binary data that came back in the response, even though the `Content-Length` header confirms its size.

**Transport Layer.** Firewalls track source and destination ports and basic flags (SYN, ACK) but omit finer detail like sequence numbers. This matters for detecting something like session hijacking, where an attacker injects a packet using a sequence number that does not follow the legitimate conversation. The example in the room shows a normal TCP handshake and data transfer, followed by a sixth packet from a different IP address with a wildly out of sequence number, which is the injection attempt. A port level firewall log would not catch that.

**Internet Layer.** Standard logs capture source and destination IPs and TTL, but miss fragment offset and total length fields. That is a problem because attackers can craft overlapping fragment offsets to disrupt packet reassembly, which is a known technique for slipping past an IDS. The room's example shows three UDP fragments where the third one's offset overlaps the second one's payload space, causing a reassembly failure (visible as an ICMP "fragment reassembly time exceeded" message).

**Link Layer.** Basic logs show source and destination MAC addresses, but nothing about how those addresses are being used across the network. Structural anomalies (a MAC address suddenly answering for multiple IPs, or an excess of gratuitous ARP replies) are invisible unless you are looking at the ARP conversation itself. This is the layer where ARP poisoning happens, and it is the topic the Wireshark: Packet Operations room comes back to in more depth.

The pattern is consistent across every layer: the header fields that make it into a standard log are enough to say something happened, but not enough to say what it means. Full packet visibility is what closes that gap.

### Sources and Flows

Once the "why traffic analysis matters" case is made, the room gives a practical way to categorize traffic in a real network, which is useful for scoping an investigation instead of drowning in every packet on the wire.

**By source:**
- **Intermediary sources** are devices that traffic passes through rather than originates from: firewalls, switches, proxies, IDS/IPS, routers, access points. They generate comparatively little traffic of their own, mostly routing protocols (EIGRP, OSPF, BGP), management protocols (SNMP, ping), logging (syslog), and supporting protocols (ARP, STP, DHCP).
- **Endpoint sources** are where traffic actually originates and terminates: servers, hosts, IoT devices, printers, cloud resources, mobile devices. These consume the bulk of network bandwidth, and this is usually where an investigation's real evidence lives.

**By flow:**
- **North-South (NS) traffic** crosses the firewall between the LAN and the WAN. It is the traffic that is most heavily monitored by default, made up of client-server protocols like HTTPS, DNS, SSH, VPN, SMTP, and RDP, each with distinct inbound and outbound streams.
- **East-West (EW) traffic** stays entirely inside the corporate LAN (including LAN extensions into cloud environments). This is often monitored less closely, which is exactly why it matters for detecting lateral movement during a compromise. It covers things like Kerberos/LDAP authentication traffic, SMB file share access, internal DNS, database connections, and backup replication traffic.

Understanding this split is less about memorizing categories and more about knowing where to look first. An external facing attack shows up in NS traffic. A compromise that is already inside the network and spreading shows up in EW traffic, which is exactly the traffic most environments log the least.

---

## 2. Wireshark: The Basics

This room is where the fundamentals shift from network theory into the actual tool. Before touching filters, the room introduces Wireshark's Statistics menu, which is the starting point for any investigation, not an afterthought. The idea is to build a hypothesis from a summary of the capture before diving into individual packets.

### The Statistics Menu

- **Resolved Addresses** lists IP addresses and hostnames pulled from DNS answers in the capture, useful for a quick look at what resources were accessed.
- **Protocol Hierarchy** breaks the capture down into a tree of protocols by packet count and percentage, which is a fast way to spot what is actually running on the wire (and to right click straight into a filter for anything unusual).
- **Conversations** shows traffic between two specific endpoints, across Ethernet, IPv4, IPv6, TCP, and UDP views.
- **Endpoints** is similar but shows unique endpoints rather than pairs, and can resolve MAC address vendor prefixes to a manufacturer name for known vendors.
- **IPv4 / IPv6 Statistics**, **DNS Statistics**, and **HTTP Statistics** each narrow the same idea (packet counts and percentages) down to a specific protocol, useful for scoping how much DNS or HTTP traffic exists before filtering into it directly.

The reason this matters for an analyst: opening a capture file and immediately typing filters is working blind. Statistics gives a shape to the data first, so the filters that come next are informed rather than guessed.

### Capture Filters vs. Display Filters

Wireshark has two filter types that look similar but do different jobs, and mixing them up is a common early mistake.

- **Capture filters** decide what gets recorded in the first place. They are set before capture starts and cannot be changed mid-capture. If a capture filter does not match the traffic pattern you actually needed, that traffic is gone. This makes capture filters higher stakes and something that needs to be understood properly before use in a live environment. Capture filter syntax uses scope (host, net, port), direction (src, dst), and protocol (tcp, udp, arp, and so on). Example: `tcp port 80`.
- **Display filters** narrow down what you are looking at within traffic that has already been captured, and can be changed freely while reviewing. Example: `tcp.port == 80`.

In practice, the safer default is to capture everything and filter down afterward with display filters, since a bad capture filter cannot be undone after the fact.

### Comparison and Logical Operators

Display filters support standard comparison operators (`==`, `!=`, `>`, `<`, `>=`, `<=`), which also work with hex values (`ip.ttl >= 0xFA` is the same as `ip.ttl >= 250`). For combining conditions, Wireshark supports `and`/`&&`, `or`/`||`, and `not`/`!`. One detail worth calling out: using `!=` for fields that can appear multiple times in a single packet can produce inconsistent results, so the recommended style is to wrap the condition in `!()` instead, for example `!(ip.src == 10.10.10.222)` rather than `ip.src != 10.10.10.222`.

### Advanced Filtering

A few functions extend basic filters for more specific searches: `contains` for a case sensitive substring match, `matches` for regex, `in` for set membership (`tcp.port in {80 443 8080}`), and `upper`/`lower`/`string` for normalizing values before comparing them. The full filter reference for all of these lives in the companion cheatsheet rather than repeated here.

### Profiles, Bookmarks, and Buttons

Wireshark supports saving filters as bookmarks or one-click filter buttons, and supports full configuration profiles so that coloring rules and saved filters do not need to be rebuilt for every new case. This is more of a workflow efficiency feature than an analysis technique, but it matters once you are running the same categories of investigation repeatedly.

---

## 3. Wireshark: Packet Operations

This room moves from "how to use the tool" into "what does an actual attack look like inside a capture." Two techniques are covered: identifying Nmap scan types by their traffic pattern, and detecting ARP poisoning.

### Recognizing Nmap Scans in a Capture

The reasoning behind covering this is straightforward: Nmap is one of the most common tools used for network reconnaissance, so being able to recognize its traffic patterns (rather than just seeing "a lot of SYN packets" and shrugging) is a baseline SOC skill.

**TCP Connect scans** (`nmap -sT`) complete the full three way handshake, since this is the only scan type available to a non-privileged user. An open port looks like a completed handshake followed by a second handshake attempt and a `RST, ACK` teardown. A closed port just gets a `RST, ACK` back immediately. Because the handshake completes, these scans typically carry a window size larger than 1024 bytes.

**TCP SYN scans** (`nmap -sS`) never complete the handshake, since they only need privileged access and are designed to be fast and quiet. An open port answers `SYN, ACK` and then gets an `RST` back from the scanner instead of a completed handshake. A closed port responds `RST, ACK` immediately, same as a connect scan. Because the handshake never completes, window size is typically 1024 bytes or less, which is the practical way to tell SYN scans apart from connect scans in a capture.

**UDP scans** (`nmap -sU`) don't use a handshake at all. An open port gives no response. A closed port returns an ICMP Type 3, Code 3 (destination unreachable, port unreachable) error. The lack of response on open ports is what makes UDP scanning slower and less reliable than TCP scanning in general, and that same trait is what an analyst uses to recognize it in a capture.

The practical value here isn't memorizing three flag combinations. It's being able to look at a burst of similar looking SYN packets and correctly separate "someone's Nmap scan" from "something else," which is a distinction that decides whether an alert gets escalated or closed.

### ARP Poisoning Investigation

ARP itself has no authentication built in and isn't a routable protocol, it only works on the local network. That lack of authentication is exactly what makes ARP poisoning possible: an attacker can send unsolicited ARP replies claiming to own an IP address that isn't theirs, and because there is no verification step, other devices on the network will simply accept the claim and update their ARP tables.

The room walks through a realistic detection process rather than presenting the answer up front, which is worth reconstructing as an investigation timeline rather than a static conclusion:

**Step 1: Spot the conflict.** Wireshark's expert info tab flags duplicate ARP responses, but only shows the second (conflicting) occurrence, not both sides of the story. Working backward from that flag, the capture shows one IP address (192.168.1.1, a likely gateway address based on the addressing) being claimed by two different MAC addresses: `50:78:b3:f3:cd:f4` and `00:0c:29:e2:18:b4`.

**Step 2: Confirm it's not a one off.** The MAC address ending in `b4` also claims the gateway's IP directly, and then goes further, crafting ARP requests against a whole range of IPs on the subnet. A single conflicting reply could be a misconfiguration. A MAC address flooding requests across an IP range and impersonating the gateway is a poisoning attempt, not an accident.

**Step 3: Correlate against real traffic.** Looking at HTTP traffic in isolation, at the IP layer everything looks normal, no link back to the ARP findings at all. This is the step that actually confirms impact: adding the MAC address column to the packet list shows that the MAC ending in `b4` is the destination for HTTP traffic that should be going to the real gateway. That is the proof of an active man in the middle, not just a suspicious ARP conflict.

**Investigation summary:**

| Role | MAC Address | IP Address | Note |
|------|-------------|------------|------|
| Attacker | `00:0c:29:e2:18:b4` | 192.168.1.25 | Real address, generated the ARP noise |
| Router/Gateway | `50:78:b3:f3:cd:f4` | 192.168.1.1 | Legitimate gateway, impersonated by the attacker |
| Victim | `00:0c:29:98:c7:a8` | 192.168.1.12 | Traffic silently redirected through the attacker |

The lesson from this walkthrough isn't really about ARP specifically. It's that real investigations rarely hand you a clean, pre-labeled answer. The duplicate ARP warning was just a starting thread. Getting from "here's a conflict" to "here's a confirmed MITM with a named victim" took correlating three separate pieces of evidence (the ARP conflict, the flooding pattern, and the HTTP destination MAC) rather than trusting any single one.

### Identifying Hosts and Users

The last part of this room covers three protocols useful for figuring out who and what is actually on the network during an investigation, which matters for deciding where to start looking and who to notify.

- **DHCP** (`dhcp` or `bootp` filter) hands out IPs and, more usefully for an investigation, DHCP Request packets carry the hostname in Option 12, while DHCP ACK/NAK packets confirm or deny the lease and can carry rejection reasons worth reading in full rather than filtering past.
- **NetBIOS (NBNS)** (`nbns` filter) queries can reveal hostnames, TTLs, and IPs for older Windows style name resolution.
- **Kerberos** (`kerberos` filter) is the default authentication protocol in Windows domains, and the `CNameString` field carries either a username or a hostname. The practical trick is that hostnames end in `$` and usernames don't, so filtering out anything containing `$` isolates real user accounts from machine accounts.

Enterprise networks tend to follow predictable naming patterns for hosts and users, which cuts both ways: it makes identification easier for a legitimate analyst, but it also makes it easier for an attacker to blend in by mimicking that same pattern once inside the network.

---

## 4. Wireshark: Traffic Analysis

The final room applies everything so far to actual threat hunting: tunneling, cleartext protocol abuse, and a couple of named attack patterns (user agent anomalies and Log4j).

### Tunneling: ICMP and DNS

Tunneling (also called port forwarding) is normally a legitimate technique for securely moving data across network segments. Attackers abuse the same idea to bypass security perimeters, specifically by hiding inside protocols that are trusted and rarely blocked, like ICMP and DNS.

**ICMP tunneling** is harder for attackers to pull off cleanly, since custom ICMP packets are often blocked by default or require elevated privileges to craft, but it does happen, usually appearing after malware execution or exploitation. The giveaways are a large volume of ICMP traffic or packets with an anomalous size. A simple starting filter is `data.len > 64 and icmp`, since attackers matching the normal 64 byte ICMP size exactly is possible but a noticeably larger payload is a fast tell.

**DNS tunneling** is the more common of the two, since DNS is essential, trusted, and frequently allowed straight through network perimeters without inspection. The tell is DNS queries that are longer than a normal lookup and use subdomains that are actually encoded commands, for example `encoded-commands.maliciousdomain.com`. Useful filters include checking query length (`dns.qry.name.len > 15 and !mdns` to exclude local link noise) and known tool signatures like `dnscat`.

### Cleartext Protocol Analysis: FTP and HTTP

**FTP** is built for simplicity, not security, which makes it a common source of MITM exposure, credential theft, and data exfiltration when used unencrypted. Response codes fall into recognizable ranges: 2xx for connection status (`227` is entering passive mode), 3xx for authentication status (`230` is a successful login, `530` is a failed one). This makes brute force and password spraying patterns easy to spot directly from response codes, for example filtering `ftp.response.code == 530` to list every failed login attempt, or combining a failed code with a specific username to see if one account is being targeted.

**HTTP** is the backbone of most web traffic and, being cleartext and rarely blocked, is a common vector for phishing pages, web attacks, data exfiltration, and C2 traffic. Standard response codes (200, 401, 403, 404, 500, 503) and request methods (GET, POST) are the starting point, but the more investigation relevant fields are things like the requested URI, the server header, and the user agent string.

### User Agent Analysis

The user agent field is a useful signal specifically because attackers can and do fake it, so it can never be trusted on its own or whitelisted. What makes it useful anyway is spotting things that look almost right but aren't: a host suddenly sending a different user agent than it normally does, a non-standard or clearly custom string, subtle misspellings of common browser identifiers (`Mozilla` vs `Mozlila`), or audit tool signatures showing up directly in the field (sqlmap, Nmap, Wfuzz, Nikto). None of these are proof on their own. They're a reason to keep looking.

### Log4j

Because the Log4Shell vulnerability has a well documented exploitation pattern, hunting for it comes down to knowing what to search for ahead of time rather than reasoning it out live. The pattern is a POST request containing the strings `jndi:ldap` or `Exploit.class`, searchable directly at the frame or IP level, plus checking user agent strings for suspicious characters like `$` or `==` that can indicate an obfuscated payload.

### HTTPS and TLS

HTTPS protects against spoofing, sniffing, and interception using TLS encryption, and without the correct decryption key, the payload is genuinely unreadable, not just inconvenient to read. This cuts both ways for a SOC analyst: it protects legitimate traffic, but it also means attackers increasingly use HTTPS for their own traffic specifically because it is trusted and hard to inspect. Recognizing a TLS handshake in a capture (`tls.handshake.type == 1` for the client hello, `type == 2` for the server response) is the baseline skill here. Actually decrypting the payload requires having the correct key material, which is a separate and more advanced topic than this room goes into.

---

## Closing Thoughts

The thread across all four rooms is the same one that opened the chapter: a log tells you something happened, a full packet capture tells you what it actually was. Nmap scan detection, ARP poisoning, DNS tunneling, and Log4j hunting are all different specific applications of that same principle, recognizing a pattern in the traffic itself that a log summary would never surface. The ARP poisoning walkthrough in particular is a good example of how these investigations actually unfold in practice: not a single smoking gun, but multiple weak signals that only add up to a confirmed finding once they are correlated against each other.
