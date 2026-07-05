# SOC Level 1

---

## Overview

SOC Level 1 introduces the role of a Junior Security Analyst — what the job actually looks like day to day, the team structure that supports that role, and where a SOC fits within the broader security hierarchy of an organization. The room pairs this theory with a short hands-on simulation of investigating and escalating a malicious IP alert.

---

## Being a Security Analyst

Working on the defensive frontline means constantly learning — during a busy shift, an analyst can be buried under a mountain of tickets (alerts and tasks that need timely resolution). The job is demanding, but it is also rewarding: stopping a real threat before it damages the organization, and seeing firsthand how attacks reported in the news actually happen in practice, both add meaning to the work.

---

## The SOC Team

A Junior Analyst does not work in isolation — a full SOC team supports the role:

| Role                   | Responsibility                                                                                                                                             |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Senior Analyst**     | Assists the junior analyst when something is unclear and handles complex cases after the initial triage is done                                            |
| **SOC Engineer**       | Maintains security tools and configures alerts to make the analyst's job easier — does not work shifts or analyze alerts directly                          |
| **SOC Manager**        | Reports the SOC team's results to top management                                                                                                           |
| **Incident Responder** | Steps in only when a major incident occurs — not part of day-to-day operations, but a signal that something serious has happened when they do get involved |

---

## Security Hierarchy

At a department level, security work is generally split across three areas:

- **Red Team** — offensive security
- **GRC** — Governance, Risk, Compliance
- **Blue Team** — defensive security

### Security Operations Center (SOC)

The SOC itself is tiered:

| Tier           | Function                                                                             |
| -------------- | ------------------------------------------------------------------------------------ |
| **L1 Analyst** | Triages alerts, escalates complex cases to L2 — this is where most SOC careers start |
| **L2 Analyst** | Investigates more advanced attacks                                                   |
| **Engineer**   | Configures and maintains security tools like EDR or SIEM                             |
| **Manager**    | Manages the SOC team as a whole                                                      |

The SOC acts as the first line of defense and handles the majority of day-to-day attacks.

### Cyber Incident Response Team (CIRT)

When SOC expertise isn't enough, or an incident spirals out of control, the CIRT (also called CSIRT or CERT) gets called in. Members of this team are expected to have broad cyber threat knowledge and are able to handle breaches without depending on tools like EDR or SIEM — the job is stressful and high-responsibility, but rewarding. Real-world examples include:

- **JPCERT** — Japan's national CERT, handling nation-wide breaches
- **Mandiant** — a private team responding to global cyber incidents
- **AWS CIRT** — investigates security incidents affecting AWS customers

### Specialized Defensive Roles

Larger companies and government agencies often split Blue Team work into more specialized, deep-knowledge roles:

- **Digital Forensics Analyst** — uncovers hidden threats in disk and memory
- **Threat Intelligence Analyst** — gathers data on emerging threat groups
- **AppSec Engineer** — maintains a secure software development lifecycle
- **AI Researcher** — studies AI-specific threats and how to defend against them

---

## Hands-On: A Day in the Life of a Junior Security Analyst

The room's practical component is a simulated SIEM environment where a suspicious IP flagged by alerts is investigated using **IP Hunter**, a tool that checks IP reputation against open-source threat databases such as **AbuseIPDB** and **Cisco Talos Intelligence** — the same kind of lookups real analysts use during alert triage.

The IP in question (`221.181.185.159`) came back flagged as malicious, tied to Port Scan, C2 Server, and PlugX activity from an ISP based in China. The alert was escalated to the appropriate staff member, and the IP was then blocked at the firewall to close out the incident. Lab completed end-to-end.

---
