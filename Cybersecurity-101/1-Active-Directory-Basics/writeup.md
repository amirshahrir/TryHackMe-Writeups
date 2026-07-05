# Active Directory Basics

## Overview

Active Directory Basics introduces the core concepts behind Microsoft's directory service — the backbone of identity and access management in most enterprise Windows environments. The room covers how domains are structured, how users and computers are organised, how policies are applied, and how authentication works across a network.

---

## What is Active Directory?

Active Directory (AD) is a centralised system used to manage users, computers, and resources across a Windows domain. Instead of managing each machine individually, administrators use AD to control everything from one place — who can log in, what they can access, and what policies apply to their machines.

---

## Core Concepts Covered

### Windows Domain
A Windows domain is a network of computers and users managed under a single Active Directory environment. It allows centralised authentication and policy enforcement across all joined machines.

### Domain Controller (DC)
The Domain Controller is the server that runs Active Directory. It handles authentication requests, stores the directory database, and enforces policies across the domain. It is the most critical and most targeted asset in any AD environment.

### Users
Users in AD can represent two things — people (employees, administrators) or services (accounts used by applications to interact with the domain). Both are treated as security principals with assignable permissions.

### Organisational Units (OUs)
OUs are containers used to organise users, computers, and groups within the domain. Their primary purpose is to apply Group Policies to specific subsets of the domain rather than applying one policy to everyone.

### Security Groups
Security Groups are used to grant permissions over resources. Rather than assigning permissions to individual users, permissions are assigned to a group and users are added to that group.

| Security Group | Description |
|----------------|-------------|
| Domain Admins | Full administrative privileges over the entire domain including all DCs |
| Server Operators | Can administer Domain Controllers but cannot change administrative group memberships |
| Backup Operators | Can access any file regardless of permissions — used for data backups |
| Account Operators | Can create or modify other accounts in the domain |
| Domain Users | Includes all existing user accounts in the domain |
| Domain Computers | Includes all existing computers joined to the domain |
| Domain Controllers | Includes all existing Domain Controllers in the domain |

### Group Policy Objects (GPOs)
GPOs are collections of settings that can be applied to OUs, users, or computers across the domain. They are distributed and enforced by the Domain Controller and are a primary tool for both configuration management and security hardening.

### Authentication — Kerberos & NetNTLM

**Kerberos** is the default authentication protocol in modern AD environments. The process involves:
- **KDC (Key Distribution Center)** — runs on the Domain Controller and issues tickets
- **TGT (Ticket Granting Ticket)** — issued to a user upon login, used to request access to services without re-entering credentials
- **Session Key** — used to authenticate the user to a specific service

**NetNTLM** is an older challenge-response authentication protocol still supported for legacy compatibility. It is considered less secure than Kerberos and is a common target in attacks such as pass-the-hash and NTLM relay.

### Trees, Forests & Trusts
- **Tree** — multiple domains sharing the same namespace, linked in a hierarchy
- **Forest** — a collection of one or more trees sharing a common schema and configuration
- **Trust** — a relationship between domains or forests that allows users in one domain to access resources in another

---

## Hands-On — Managing Users and Computers via GUI

The room involved practical use of **Active Directory Users and Computers** and **Group Policy Management** through the GUI:

1. Opened **Active Directory Users and Computers** on the Domain Controller
2. Navigated the OU structure — viewed how users and computers were organised
3. Created and modified user accounts — assigned users to security groups
4. Opened **Group Policy Management** — reviewed existing GPOs linked to OUs
5. Explored how GPO distribution flows from the DC down to linked OUs

---

## Connection to Windows Fundamentals

Active Directory builds directly on top of concepts from Windows Fundamentals. User Accounts, UAC, and Computer Management all exist within the context of a standalone Windows machine — AD scales those same concepts across an entire network. Event Viewer becomes even more critical in a domain environment, as DC logs are a primary source for detecting attacks like brute force, lateral movement, and privilege escalation.
