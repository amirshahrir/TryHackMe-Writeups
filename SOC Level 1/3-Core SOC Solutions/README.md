# Core SOC Solutions

---

## Overview

Core SOC Solutions covers four of the main tool categories a SOC analyst works with day to day: Endpoint Detection and Response (EDR), Splunk, the ELK Stack, and SOAR. Rather than teaching one tool in isolation, the room builds a picture of how a modern SOC operates across detection, investigation, visualization, and response, and where each tool fits into that picture.

The EDR, ELK, and SOAR sections below are concept focused. The Splunk section includes a hands-on lab and gets the most depth, since that is where the actual investigative work happened.

---

## Endpoint Detection and Response (EDR)

### Why EDR Exists

Most business operations run on digital devices, and remote work has pushed many of those devices outside the traditional network perimeter. EDR exists to give security teams visibility and control over endpoints regardless of where they physically sit.

### The Three Pillars of EDR

**Visibility**: EDR collects data from endpoints, including process activity, registry changes, file modifications, and user actions. Analysts can view this as a process tree (a graphical map of parent and child processes) and use it for historical threat hunting.

**Detection**: EDR combines signature based detection with behavior based detection. This matters because attackers increasingly craft malware to look clean and abuse legitimate processes. A common example is `winword.exe` spawning `powershell.exe`, an unusual parent child relationship that would get flagged even though both processes are legitimate on their own. EDR also supports custom IOC feeds and maps detections to MITRE ATT&CK tactics and techniques.

**Response**: From a single console, an analyst can isolate an infected endpoint, terminate a malicious process, quarantine a file, or connect remotely to the host to run further commands. Isolating a host is particularly important for stopping lateral movement before it spreads across the network.

### Limitation Worth Noting

EDR is host only. It has no visibility into network level threats, which is part of why it is usually paired with other tools like a SIEM.

### Market Context

Common EDR platforms include CrowdStrike Falcon, SentinelOne ActiveEDR, Microsoft Defender for Endpoint, and Symantec EDR. CrowdStrike's Real Time Response (RTR) console is a specific example of the remote access response capability.

---

## Splunk

Splunk is one of the most widely deployed SIEM (Security Information and Event Management) platforms in the industry. Where EDR watches individual endpoints, a SIEM like Splunk pulls logs from across an entire environment (endpoints, servers, databases, web traffic) into one place so they can be searched and correlated. That is the concept that connects this section back to EDR: EDR gives depth on a single host, Splunk gives breadth across the whole environment.

### How Splunk Is Built

Splunk's architecture has three main pieces:

- **Forwarder**: A lightweight agent installed on the monitored endpoint. It collects log data (Windows Event Logs, Sysmon data, web server logs, database logs, and so on) and ships it to the central Splunk instance without putting much load on the endpoint itself.
- **Indexer**: Takes the raw incoming data and parses it into structured field value pairs, then stores it so it can be searched efficiently.
- **Search Head**: The interface analysts actually use. Queries are written in SPL (Search Processing Language) and results can be shaped into tables or visualizations like pie and bar charts.

### Where Splunk Sits Among Alternatives

Splunk is not the only option in this space. Microsoft Sentinel is the cloud native equivalent inside Azure and Microsoft 365 environments. IBM QRadar bundles SIEM with SOAR and behavioral analytics. Google Chronicle focuses on fast log normalization at scale, and Exabeam leans heavily on user and entity behavior analytics (UEBA). On the open source side, Elastic Security (built on the ELK stack, covered below) and Grafana with Loki are common cost effective alternatives. Knowing Splunk's concepts (forwarding, indexing, search) transfers reasonably well to any of these, since they all solve the same core problem in similar ways.

### Hands-On Lab: VPN Log Investigation

The lab task was to ingest a VPN log file and answer investigative questions using Splunk's search interface.

**Getting the data in**: The first step was uploading `VPN_logs.json` through Splunk's Add Data workflow and creating a new index named `VPN_Logs` to hold it. This is the same forwarder-to-indexer concept from the architecture above, just done manually through the UI instead of through a live forwarder agent.

**Working through the questions**: Once the data was indexed, the investigation moved through a series of questions, each one narrowing down the log data using a different field or condition:

- Total number of events in the ingested log file
- Number of events tied to a specific user (Maleena)
- Identifying which username was associated with a given source IP address
- Counting events that originated from all countries except one (France), which relies on excluding a value rather than matching one
- Counting events tied to a second specific IP address

Each of these maps to a different way of narrowing a search: filtering by a single field value, matching a specific field to a specific value, and excluding a value using a NOT-style condition. The specific search syntax used to get each answer is intentionally left out of this write-up, since the goal here is documenting the investigative logic rather than reproducing exact commands.

### Personal Notes

Splunk was the first real SIEM-style platform I have worked in, and my first instinct was to compare it to a CMS, since it has that same "a lot of screens and options until you know where things live" feel. The friction wasn't the underlying logic (filtering, searching, excluding values are concepts I already understood), it was just not yet knowing where those actions live in Splunk's interface.

Given how central Splunk is to SOC work, this is one I want to go back and spend more time with to actually get comfortable with the concepts, not just the specific steps in this lab. It should also make platforms like Microsoft Sentinel easier to pick up later, since the underlying SIEM concepts (forwarding, indexing, field based search) carry over even when the interface changes.

---

## ELK Stack (Elasticsearch, Logstash, Kibana)

ELK is an open source alternative to a platform like Splunk, and it is common enough in SOC environments that some teams use it as their SIEM.

It is made up of four components working together: **Beats** (lightweight agents that ship data from endpoints, similar in role to a Splunk forwarder), **Logstash** (processes and normalizes that data through input, filter, and output stages), **Elasticsearch** (a full text search engine that stores and indexes the normalized data), and **Kibana** (the web interface used to search, visualize, and build dashboards from that data).

Inside Kibana, most of an analyst's time is spent in the **Discover** tab, searching raw logs using KQL (Kibana Query Language). One clarification worth noting: KQL free text search matches on individual terms, not just full phrases. Searching for a single word like `United` will still return documents that contain that word, including ones where it appears as part of a longer phrase like "United States." It is quoted phrase searches (`"United States"`) that require an exact match, not single unquoted terms. KQL also supports wildcards (`United*`) and logical operators (AND, OR, NOT) to combine or exclude terms.

Kibana's Visualization and Dashboard tabs let analysts turn saved searches into charts and combine multiple visualizations into a single view, which is useful for things like ongoing VPN log monitoring.

---

## SOAR (Security Orchestration, Automation, and Response)

### The Problem SOAR Solves

A traditional SOC juggles several disconnected tools (SIEM, EDR, firewall, IAM, ticketing) and investigating a single alert often means manually switching between all of them. Combined with alert fatigue and undocumented, tribal-knowledge-based investigation processes, this slows SOC teams down significantly.

### What SOAR Adds

SOAR unifies those tools into a single interface and introduces **playbooks**, which are predefined, branching investigation workflows. A VPN brute force playbook, for example, might check the SIEM for whether the user normally logs in from that IP, check threat intelligence for the IP's reputation, check for successful logins, and escalate to containment if needed.

The key difference between orchestration and automation is that orchestration defines the workflow, while automation lets SOAR execute that workflow's steps itself (querying the SIEM, checking IP reputation, disabling a user in the IAM, opening a ticket) without an analyst manually clicking through each tool.

---
