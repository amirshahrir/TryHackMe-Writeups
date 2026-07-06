# Cyber Defense Frameworks

---

## Overview

This chapter covers the frameworks that security professionals use to describe, structure, and communicate about how attackers operate. Understanding these frameworks matters because they give defenders a shared language for describing an intrusion, comparing it to known attacker behavior, and building detections around it. The frameworks covered here are the Cyber Kill Chain, the Unified Kill Chain, Threat Modelling, MITRE ATT&CK, and D3FEND.

---

## The Cyber Kill Chain

### Origins

The Cyber Kill Chain concept borrows from a military model, where an attack is broken into identifying a target, deciding to strike it, and destroying it. Lockheed Martin adapted this military thinking into a cybersecurity specific framework in 2011. The core idea is that an attacker has to succeed at every single stage to reach their objective, which means a defender only has to break one link in the chain to stop the attack entirely.

### Why It Matters

Studying the Kill Chain helps organizations recognize ransomware attacks, breaches, and Advanced Persistent Threats (APTs) earlier, and helps identify which security controls are missing at each stage. For SOC analysts, threat hunters, and incident responders, it provides a structured way to map what stage an intrusion is at and what the attacker is likely trying to do next.

### The Seven Phases

| Phase                    | What Happens                                                                                                                                                                                                                                             |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Reconnaissance           | The attacker researches the target, often passively (WHOIS lookups, social media, OSINT) or actively (port scanning, social engineering). Good recon leads to a more convincing and effective attack.                                                    |
| Weaponization            | Raw information from recon is turned into an actual attack tool, such as a malicious document, an infected USB drive, or a phishing kit. Some attackers buy ready made payloads, others (often APT groups) build custom malware to avoid detection.      |
| Delivery                 | The payload is transmitted to the target, commonly through phishing email, physical USB drops, or watering hole attacks that compromise a site the victim already visits.                                                                                |
| Exploitation             | The payload actually executes, taking advantage of a vulnerability. This can be a known CVE, a zero day, or something as simple as a user enabling a malicious macro.                                                                                    |
| Installation             | The attacker sets up a way to keep access to the system, for example a web shell, a backdoor, a modified Windows service, or a registry run key, so they can return even if the initial connection is lost.                                              |
| Command and Control (C2) | The compromised host starts communicating back to infrastructure the attacker controls, often called beaconing. Modern C2 tends to hide inside normal looking HTTP, HTTPS, or DNS traffic rather than older methods like IRC, which is easy to spot now. |
| Actions on Objectives    | With hands on access to the system, the attacker does whatever they came to do: stealing credentials, escalating privileges, moving laterally, exfiltrating data, or destroying backups and data to block recovery.                                      |

---

## The Unified Kill Chain

The Unified Kill Chain expands on the original seven phase model with eighteen phases, adding steps such as Social Engineering, Defense Evasion, Pivoting, Discovery, Privilege Escalation, Credential Access, Collection, and Exfiltration as their own distinct stages. The extra granularity gives a closer match to how modern attacks and APT campaigns actually unfold, since a real intrusion rarely moves through the seven original phases in a single clean pass.

A closely related idea introduced here is persistence: a malicious foothold designed to survive things like a reboot, for example malware that reinserts itself through a startup registry key.

---

## Threat Modelling

Threat modelling is the process of figuring out what in an organization actually needs protecting and how it could realistically be attacked, so that resources go toward the highest risk gaps instead of being spread thin. The general process is:

1. Identify the systems and applications in scope, and decide how critical or sensitive each one is.
2. Assess what vulnerabilities those systems have and how they could be exploited.
3. Plan and implement actions to close those gaps.
4. Put policies in place so the same issues do not resurface, for example rolling out a Secure Development Lifecycle or running phishing awareness training.

Frameworks like STRIDE, DREAD, and CVSS give structure to the vulnerability assessment part of this process, and the Unified Kill Chain itself can help identify what attack surfaces exist in the first place.

---

## MITRE ATT&CK

### Core Idea

MITRE created ATT&CK in 2013 as a public knowledge base of real world adversary behavior. It breaks attacker actions into three layers:

- **Tactic**: the attacker's goal (the why).
- **Technique**: how that goal is achieved.
- **Procedure**: the specific implementation of that technique in practice.

What started as a Windows focused model has since grown to cover macOS, Linux, cloud, mobile, and industrial control systems.

### The Matrix

The ATT&CK Matrix lays tactics across the top with their related techniques nested underneath, and the ATT&CK Navigator is the tool used to explore and annotate it. As an example, under the Reconnaissance tactic, one technique is Active Scanning, which breaks down further into sub-techniques like scanning IP blocks or vulnerability scanning.

### Why Different Roles Use It

The framework's real value is that it gives everyone the same vocabulary instead of ten different names for the same attacker action:

- CTI teams map observed threat actor behavior into reusable, shareable profiles.
- SOC analysts use it to add context to alerts and prioritize what to triage first.
- Detection engineers map their SIEM and EDR rules against it to spot coverage gaps.
- Incident responders use it to lay out a timeline of an intrusion in a consistent way.
- Red and purple teams build realistic attack simulations around known techniques.

### Cyber Analytics Repository (CAR)

CAR is MITRE's companion project of ready made detection analytics, built directly on top of ATT&CK. Each entry explains the reasoning behind a detection and often includes example queries for tools like Splunk, which helps translate a technique on paper into something an analyst can actually search for in their own environment.

---

## D3FEND

Where ATT&CK documents the attacker's side, D3FEND documents the defender's side, using the same kind of structured, common vocabulary approach. It stands for Detection, Denial, and Disruption Framework Empowering Network Defense, and organizes defensive techniques into seven tactics.

One example covered is Credential Rotation, which focuses on regularly rotating passwords so that credentials stolen earlier in an intrusion stop being useful to the attacker. Each D3FEND entry explains the mechanics of the defense, implementation considerations, and how it ties back to the specific ATT&CK techniques it counters, which makes it easy to go from "here is how they attacked" to "here is what stops it" in the same framework family.

---

## Personal Notes

> This whole chapter is written from the attacker's point of view, from recon all the way to actions on objectives. At first that felt a bit backwards for a defensive security chapter, but the point clicked once I connected it back to what these frameworks are actually used for. If you understand how an attacker thinks and moves through a Kill Chain, you know what to look for at each stage and can build detections or block a stage before the attacker reaches their goal. ATT&CK and D3FEND make that connection explicit, since D3FEND techniques are literally mapped back to the ATT&CK techniques they defend against. Knowing the attack path is what makes the defense make sense.
