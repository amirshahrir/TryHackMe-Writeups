# Windows Fundamentals

## Overview

Windows Fundamentals is a three-part room that walks through the Windows operating system from a security and IT administration perspective. Rather than command-line interaction, the focus is almost entirely on the GUI — navigating the built-in tools, utilities, and settings that IT professionals and security analysts rely on day to day.

- **Part 1** covers the desktop environment, the file system, User Account Control, Control Panel, Settings, and Task Manager
- **Part 2** covers administrative utilities such as System Configuration, Computer Management, and Resource Monitor
- **Part 3** covers Windows Security — the built-in security dashboard and its components

---

## Mindset Shift Coming from Linux

Coming from Linux Fundamentals, the biggest shift was moving away from the terminal entirely. Where Linux relies heavily on commands and the file system structure, Windows is designed around GUI navigation — finding information means knowing where to click rather than what to type. The challenge is not syntax but orientation: knowing which tool lives where and what it actually controls under the hood.

---

## Tools & Utilities Covered

The GUI tools and utilities interacted with across the three parts:

- Windows Properties
- System32
- User Accounts
- Settings
- Task Manager
- System Configuration
- System Properties
- User Account Control (UAC)
- Computer Management
- Resource Monitor
- Registry Editor
- Task Scheduler
- Event Viewer
- Windows Security Dashboard

---

## Standout Tool — User Account Control (UAC)

UAC stood out the most. In a typical office environment, UAC prompts are something end users see but rarely think about — it is the IT team that interacts with it meaningfully to grant or restrict access to shared folders and system-level resources. Going through this room made it clear that UAC is not just an annoyance prompt but an actual privilege escalation boundary. Understanding how it works matters both for defenders configuring it correctly and attackers looking to bypass it.

---

## Walkthrough — Windows Security Dashboard

One of the more hands-on sections involved navigating the Windows Security dashboard and exploring each of its built-in protection areas:

1. Opened **Windows Security** from the Start Menu
2. Reviewed **Virus & Threat Protection** — checked protection settings and scan options
3. Navigated to **Firewall & Network Protection** — reviewed active profiles (Domain, Private, Public)
4. Explored **App & Browser Control** — reviewed SmartScreen settings for apps and web browsing
5. Checked **Device Security** — reviewed core isolation and security processor details

This section highlighted how much security functionality is built directly into Windows that many users never actively engage with.

---

## Additional Notes

**Task Scheduler** — allows the creation and management of automated tasks that run applications or scripts at specified times, at log in/off, or on a recurring schedule. Relevant for both legitimate automation and persistence techniques used by attackers.

**Event Viewer** — logs all events that occur on the system, functioning as an audit trail. Used by analysts to diagnose issues and investigate suspicious activity. A critical tool for blue team work.

**Crash Dump / Startup and Recovery** — accessible through Advanced System Settings, this allows configuration of how Windows handles critical failures such as a Blue Screen of Death. Crash dump files are valuable for post-incident analysis.

---

## Key Takeaway

Windows is the dominant OS in enterprise environments. Understanding its built-in tools is foundational for both offensive and defensive security work — whether that means knowing where attackers look to persist, escalate privileges, or evade detection, or knowing where defenders go to monitor, audit, and respond.
