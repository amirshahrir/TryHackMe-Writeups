# Defensive Security

---

## Overview

This chapter covers the foundational knowledge behind defensive security — the practice of protecting systems, detecting threats, and responding to incidents. The rooms progress from understanding how a Security Operations Center functions, through the methodology of digital forensics, into how security incidents are classified and handled, and finally into the role logs play as the backbone of visibility across all of the above.

> **Personal Note:** This chapter was genuinely clarifying. Going through SOC structure, forensic methodology, and incident response as interconnected concepts — rather than isolated facts — made the bigger picture of how defensive security actually operates click into place. It makes the day-to-day work of a SOC analyst feel far less abstract.

---

## Security Operations Center (SOC)

### What is a SOC?

A Security Operations Center is a dedicated facility operated by a specialised team whose purpose is to continuously monitor an organisation's network and resources, identify suspicious activity, and prevent damage before it occurs. The SOC team operates **24 hours a day, 7 days a week** — security threats do not follow business hours.

---

### Purpose of a SOC

The SOC's work falls into two broad categories: **Detection** and **Response**.

**Detection:**

| Activity | Description |
| -------- | ----------- |
| Detect vulnerabilities | Identify weaknesses in software or systems that an attacker could exploit to exceed their permission level. While remediation may sit with other teams, unpatched vulnerabilities directly affect the organisation's overall security posture |
| Detect unauthorised activity | Identify when a legitimate account is being misused — for example, an attacker using stolen credentials to log in. Geographic anomalies and unusual access patterns are common indicators |
| Detect policy violations | Identify actions that breach the organisation's security policy, such as downloading pirated software or transmitting confidential files through unsecured channels |
| Detect intrusions | Identify unauthorised access to systems or networks — whether through an exploited web application or a user whose machine was infected after visiting a malicious site |

**Response:**

Once an incident is detected, the SOC supports the incident response process — helping to minimise the impact of the incident and contributing to the root cause analysis that follows.

---

### The Three Pillars of a SOC

#### People

The SOC team is structured in tiers, with each level handling a different depth of analysis:

| Role | Responsibilities |
| ---- | ---------------- |
| **SOC Analyst — Level 1** | First responders to any detection. Perform initial alert triage to determine whether an alert is harmful, and escalate through proper channels |
| **SOC Analyst — Level 2** | Handle detections that require deeper investigation. Correlate data across multiple sources to build a more complete picture |
| **SOC Analyst — Level 3** | Experienced analysts who proactively hunt for threat indicators and lead incident response activities including containment, eradication, and recovery |
| **Security Engineer** | Deploy and configure the security solutions the analyst team works with daily |
| **Detection Engineer** | Build and maintain the detection logic (rules) behind security solutions. This role is sometimes handled by Level 2 and 3 analysts, and sometimes exists as a dedicated position |
| **SOC Manager** | Manages SOC processes and team operations. Serves as the primary point of contact between the SOC and the organisation's CISO, providing regular updates on security posture |

---

#### Process

**Alert Triage** is the foundation of SOC operations — the first action taken in response to any alert. Triage determines the severity of an alert and how it should be prioritised.

A structured triage follows the **5 Ws** framework:

| Question | Purpose |
| -------- | ------- |
| **What** | What was detected? |
| **When** | At what date and time did it occur? |
| **Where** | On which system or location was it detected? |
| **Who** | Which user or account is associated with the alert? |
| **Why** | What is the likely cause or motivation behind the activity? |

**Example — Malware detected on an employee PC:**

- **What:** A malicious file was detected on a host inside the organisation's network
- **When:** File detected at 13:20 on April 27, 2026
- **Where:** Found in the directory of an employee's PC
- **Who:** Associated with the user account of that PC
- **Why:** Investigation revealed the file was downloaded from a pirated software website — the user had been attempting to obtain a free version of a paid application

**Reporting** follows triage. Findings are escalated to higher-level analysts via tickets, assigned to relevant personnel, and documented in a report that covers all 5 Ws alongside thorough analysis and supporting screenshots.

---

#### Technology

Security solutions reduce the manual effort required from the SOC team to detect and respond to threats. The most common include:

| Technology | Description |
| ---------- | ----------- |
| **SIEM** (Security Information and Event Management) | Present in almost every SOC environment. Collects logs from across the network, applies detection rules to identify suspicious activity. Provides **detection** capability only |
| **EDR** (Endpoint Detection and Response) | Operates at the endpoint level. Provides real-time and historical visibility into endpoint activity and can carry out automated responses |
| **Firewall** | Acts as the barrier between internal and external networks. Monitors and filters incoming and outgoing traffic based on defined rules |

Other technologies — including Antivirus, EPP, IDS/IPS, XDR, and SOAR — may also be deployed depending on the organisation's specific needs and available resources.

---

## Digital Forensics

### What is Digital Forensics?

Digital forensics is the discipline of collecting, examining, analysing, and reporting on digital evidence in a way that preserves its integrity and makes it admissible in legal proceedings. It applies scientific methodology to the investigation of digital devices and data.

---

### The Four Phases of Digital Forensics

**1. Collection (Acquisition)**

The goal of this phase is to identify and gather all potential evidence — PCs, laptops, USB drives, cameras, and any other relevant devices. The integrity of the original data must be protected throughout; nothing must be altered. A strict **Chain of Custody** document is maintained, detailing every item collected and every person who has handled it.

**2. Examination**

Raw evidence is rarely small. This phase filters and extracts only the relevant information from large datasets — narrowing down based on specific criteria such as a date range or a particular user account. This prevents investigators from being overwhelmed by irrelevant data.

**3. Analysis**

Evidence from multiple sources is correlated to draw conclusions. Events are reconstructed in chronological order to understand what happened, when, and how — building a coherent account of activity relevant to the case.

**4. Reporting**

Findings are communicated to non-technical stakeholders such as law enforcement or management. The report must cover the methodology used, detailed findings, and professional recommendations. An **Executive Summary** is required so that those without a technical background can understand the outcome.

---

### Types of Digital Forensics

| Type | Description |
| ---- | ----------- |
| **Computer Forensics** | The most common type. Focuses on investigating computers and the data stored on them |
| **Mobile Forensics** | Investigates mobile devices, extracting evidence such as call records, text messages, and GPS location data |
| **Network Forensics** | Extends investigation beyond individual devices to cover the entire network. Primary evidence comes from network traffic logs |
| **Database Forensics** | Investigates intrusions into databases that result in data modification or exfiltration |
| **Cloud Forensics** | Investigates data stored on cloud infrastructure. Can be challenging due to limited on-premises evidence and shared responsibility models |
| **Email Forensics** | Investigates emails to determine whether they are part of phishing campaigns, fraud, or other malicious activity |

---

### Evidence Acquisition

#### Proper Authorisation

Before collecting any data, the forensics team must obtain authorisation from the relevant authorities. Evidence collected without prior approval may be deemed inadmissible in court. Additionally, forensic evidence frequently contains private and sensitive data — proper authorisation ensures the investigation stays within legal boundaries.

#### Chain of Custody (CoC)

The Chain of Custody is a chronological, written record of every person who has had physical or digital possession of a piece of evidence. Without it, evidence is typically considered tainted and cannot be used in legal proceedings.

A CoC document tracks the following:

| Detail | Description |
| ------ | ----------- |
| **Evidence Metadata** | Name, model, serial number, and physical description of the device |
| **The Collector** | Name and badge/ID number of the person who first seized the item |
| **Temporal Data** | Exact date and time of collection or transfer |
| **Storage Path** | Specific location where the evidence is stored |
| **Access Logs** | Every person who accessed the evidence, the reason, and when it was returned |

#### Write Blockers

A write blocker is a specialised tool — hardware or software — that allows an investigator to read data from a drive without the risk of accidentally modifying it.

The problem it solves: modern operating systems constantly perform background tasks, such as updating "last accessed" timestamps or indexing files. Plugging a suspect's drive directly into a standard computer would cause the OS to automatically alter the drive's data.

A write blocker acts as a one-way gate — data flows from the suspect's drive to the forensic workstation, but nothing can travel back. This preserves the original state of the evidence and ensures that file timestamps and metadata remain exactly as they were found.

#### Metadata Extraction — Practical Examples

Digital files carry metadata that can be valuable forensic evidence — information embedded in the file itself rather than visible in its contents.

**PDF Metadata — `pdfinfo`:**

```bash
pdfinfo document.pdf
```

Returns metadata such as the document's author, creation date, modification date, and the software used to produce it.

**Image EXIF Data:**

```bash
exiftool image.jpg
```

Returns embedded metadata from a photo, which can include the camera model, GPS coordinates of where the photo was taken, the exact timestamp, and lens information. This data is embedded automatically by the device and is often overlooked by the person who took the photo.

---

## Incidents

### False Positives and True Positives

Before responding to an alert, a SOC analyst must first determine whether it represents a genuine threat.

| Classification | Description | Example |
| -------------- | ----------- | ------- |
| **False Positive** | An alert that triggers on legitimate activity | A security solution flags a large volume of outbound data transfer — investigation reveals the system was performing a scheduled backup to cloud storage |
| **True Positive** | An alert that correctly identifies a real threat | A security solution flags a phishing email — investigation confirms it was a genuine phishing attempt targeting an employee |

---

### Common Security Incidents

**Malware Infections**
Malicious software — whether delivered as a text file, document, or executable — designed to damage systems, networks, or applications. Malware-related incidents are among the most frequently encountered by SOC teams, and the severity of impact varies depending on the specific type of malware involved.

**Security Breaches**
Unauthorised access to confidential data. The primary concern is the **Confidentiality** pillar of the CIA Triad. Breaches are typically the result of deliberate external attacks or unauthorised internal access.

**Data Leaks**
The exposure of sensitive information to unauthorised parties. Unlike a breach, data leaks are often caused by human error or misconfiguration rather than deliberate attack. Attackers who obtain leaked data may use it for reputational damage or extortion.

**Insider Attacks**
Attacks initiated by someone within the organisation — such as a disgruntled employee. Insider threats carry an extremely high risk because insiders already possess greater access and institutional trust than an outside attacker would need to acquire. A classic example is an employee using a USB drive to introduce malware on their final day of employment.

**Denial of Service (DoS) Attacks**
An attack that floods a system with false or excessive requests in order to exhaust its resources. This directly targets the **Availability** pillar of the CIA Triad — legitimate users are unable to access the service because the system is overwhelmed handling illegitimate traffic.

---

## Incident Response

### The Incident Response Lifecycle

Incident response follows a structured lifecycle. Each phase builds on the one before it, ensuring that incidents are handled thoroughly rather than just patched over.

| Phase | Goal | Example |
| ----- | ---- | ------- |
| **Preparation** | Build the resources, plans, and teams needed before an incident occurs | Conducting employee awareness training to help staff identify and report phishing emails |
| **Identification** | Detect abnormal behaviour or indicators of compromise that signal an active incident | Noticing a sudden spike in data being sent from a specific machine to an unknown external server |
| **Containment** | Limit the damage and prevent the threat from spreading to other systems | Disconnecting a compromised laptop from the network so the attacker cannot move laterally to other servers |
| **Eradication** | Completely remove the threat from the environment | Running deep malware scans or deleting malicious scripts to ensure systems are clean |
| **Recovery** | Restore affected systems to normal operation | Reconfiguring a host and restoring data from a secure, verified backup |
| **Lessons Learned** | Review the incident to identify gaps and improve defences against recurrence | Holding a post-incident meeting to analyse root cause and update firewall rules accordingly |

---

### Response Techniques — Playbooks and Runbooks

Two documents guide how a SOC team executes its response. They serve different purposes and operate at different levels of detail.

**Playbook**

A playbook is a high-level document covering the entire incident lifecycle — from the moment an alert fires through to the final review. It addresses not only technical steps but also legal, communication, and management considerations.

- **Focus:** What is the goal, and who needs to be involved?
- **Contents:** Decision trees, assigned roles (lead investigator, press contact), and compliance requirements

*Example — Malware Playbook:*
1. SOC Analyst identifies a malware alert
2. Confirm whether it is a false positive
3. Notify the Department Head that a laptop is being taken offline
4. If ransomware is confirmed, engage the Insurance and Legal teams

**Runbook**

A runbook is a granular, step-by-step technical reference for a specific task. Where a playbook says "isolate the host," the runbook specifies exactly which commands to run to make that happen.

- **Focus:** How exactly is this technical task performed?
- **Contents:** Specific commands, tools to open, and configuration settings

*Example — Server Isolation Runbook (Linux):*
1. Log in to the terminal as root
2. Run `ifconfig` to identify the active network interface
3. Run `ip link set eth0 down` to sever the connection
4. Verify isolation by attempting to ping `8.8.8.8`

The relationship between the two is straightforward: the playbook governs the strategy; the runbook executes it.

---

## Logs Fundamentals

### What Are Logs and Why Do They Matter?

Logs are automatically generated records of events and activities produced by systems, applications, and network devices. Across the SOC, digital forensics, and incident response, logs serve as the primary source of visibility — without them, there is no trail to follow.

### Key Use Cases

| Use Case | Purpose |
| -------- | ------- |
| **Security Event Monitoring** | Real-time detection of anomalous or suspicious behaviour — identifying active threats as they happen |
| **Incident Investigation and Forensics** | Providing a digital paper trail that enables root cause analysis — reconstructing exactly how and why an incident occurred |
| **Troubleshooting** | Diagnosing system or application errors, helping technical teams identify and resolve failures quickly |
| **Performance Monitoring** | Tracking resource usage and system health to identify bottlenecks and optimise performance |
| **Auditing and Compliance** | Creating an official record of activity that demonstrates security policies are being followed during regulatory audits |

---

### Common Log Types

| Log Type | Primary Use | Examples |
| -------- | ----------- | -------- |
| **System Logs** | Troubleshooting OS issues and tracking core system health | Startup/shutdown timestamps, hardware failures, driver loading events |
| **Security Logs** | Detecting and investigating security incidents | Successful/failed logins, permission checks, policy changes, abnormal user behaviour |
| **Application Logs** | Tracking events specific to a piece of software | User interactions within an app, internal errors, configuration changes |
| **Audit Logs** | Maintaining a record of system changes for compliance | Who accessed sensitive data, major system modifications, evidence of policy enforcement |
| **Network Logs** | Monitoring traffic moving in and out of the network | Firewall logs, connection attempts to external IPs, inbound/outbound data volumes |
| **Access Logs** | Identifying exactly who or what accessed a specific resource | Web server visitor IPs, database queries, API request history |
