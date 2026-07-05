# OWASP Top 10 (2025)

---

## Overview

This chapter closes out the Cyber Security 101 path by looking at web application security through the lens of the **OWASP Top 10** — a list maintained by the Open Web Application Security Project that ranks the most critical security risks facing web applications today.

TryHackMe splits this material into three separate rooms, each grouping a handful of OWASP categories around a shared theme:

1. **IAAA Failures** — how breakdowns in Identity, Authentication, Authorization, and Accountability show up as real vulnerabilities (A01, A07, A09)
2. **Application Design Flaws** — mistakes baked into how a system is built or deployed, before a single line of "vulnerable" code is even written (AS02, AS03, AS06)
3. **Insecure Data Handling** — what goes wrong when applications don't protect or validate the data flowing through them (A04, A05, A08)

Rather than memorizing ten disconnected vulnerability names, the point of these rooms is to see that most web app security issues trace back to one of a few root causes: the server trusts the client too much, the environment was set up carelessly, or data isn't verified/protected at the right point.

---

## Room 1: OWASP Top 10 — IAAA Failures

### The IAAA Model

Before getting into specific vulnerabilities, this room frames everything around four concepts that any application needs to get right:

| Concept | What It Means |
| --- | --- |
| **Identity** | The unique account (username, email, service ID) that represents a person or system |
| **Authentication** | Proving that identity is genuine — passwords, OTPs, passkeys |
| **Authorization** | Defining what an authenticated identity is actually allowed to do |
| **Accountability** | Recording and alerting on who did what, when, and from where |

The three OWASP categories in this room are really just different ways one of these four pillars can fail.

### A01: Broken Access Control

Broken Access Control happens when the server doesn't properly enforce who is allowed to access what on every single request — not just at login, but on every action afterward.

The most common real-world version of this is **IDOR (Insecure Direct Object Reference)**. If an application exposes a raw identifier in a URL or request — like `?id=7` — and doesn't check whether the logged-in user is actually supposed to see that record, then simply changing the number to `?id=8` can expose someone else's data entirely.

An important distinction covered here: if changing that ID lets you view another *regular* user's data (not an admin's), you haven't gained a new privilege level — you've just accessed data outside your own scope. This specific case is called **Horizontal Privilege Escalation**, and it happens because the application trusts whatever ID the client sends instead of verifying ownership server-side.

**Hands-on exercise:** The room includes a static site with an `accountID` value sitting in the URL. The task is to manipulate that value and find the one account holding over $1 million. Changing the ID was enough — there was no check anywhere confirming that the currently logged-in session actually owned the account being requested. It's a small, deliberately obvious example, but it makes the abstract definition of IDOR concrete: the vulnerability isn't some complex exploit, it's just a missing "does this ID belong to you?" check.

### A07: Authentication Failures

This category covers situations where an application can't reliably verify — or properly bind — a user's identity. Common patterns include:

- Username enumeration (the app reveals whether a username exists, e.g. through different error messages)
- Weak or guessable passwords with no lockout or rate-limiting
- Logic flaws in login or registration flows
- Insecure session or cookie handling

The end result of any of these is the same: an attacker becomes able to log in as someone else, or a session ends up bound to the wrong account entirely.

### A09: Logging & Alerting Failures

Good logging is what makes Accountability possible in practice — without it, there's no way to prove who did what, when, or from where after the fact.

In practice, this failure shows up as missing authentication event logs, vague or unhelpful error messages, no alerting on brute-force attempts or privilege changes, short log retention windows, or logs stored somewhere an attacker could tamper with them. None of these are exotic — they're usually just logging being treated as an afterthought rather than a security control.

---

## Room 2: OWASP Top 10 — Application Design Flaws

This room's theme is that some of the most damaging vulnerabilities aren't bugs in the code at all — they're the result of how a system was configured, assembled, or architected from the start.

### AS02: Security Misconfiguration

This happens when systems, servers, or applications are deployed with unsafe defaults, incomplete settings, or unnecessary exposure — not because the code is wrong, but because of how the environment was set up.

**Common patterns:**
- Default credentials or weak passwords left unchanged
- Unnecessary services or endpoints exposed to the internet
- Misconfigured cloud storage (S3 buckets, Azure Blob, GCP buckets)
- Unrestricted API access with missing authentication
- Verbose error messages leaking stack traces or system details
- Outdated software or containers with known vulnerabilities
- Exposed AI/ML endpoints without access controls

**Prevention** centers on hardening defaults, removing unused services, enforcing least privilege, keeping software patched, hiding error details from end users, and regularly auditing cloud configurations.

### AS03: Software Supply Chain Failure

This covers weaknesses introduced not by an organization's own code, but by the third-party components, libraries, or services it depends on. Attackers target these weak links to inject malicious code or bypass security controls without ever touching the "real" application.

**Common patterns:**
- Using unverified or unmaintained dependencies
- Auto-installing updates without any verification step
- Over-relying on third-party AI models with no monitoring
- Insecure CI/CD pipelines that allow tampering
- Poor tracking of licensing or component provenance

**Protecting against it** means verifying components before use, patching dependencies regularly, signing and auditing updates, locking down CI/CD pipelines, and monitoring for unusual behavior from anything the application depends on.

### AS06: Insecure Design

Insecure design is what happens when flawed logic or architecture gets built into a system from day one — usually because threat modeling was skipped, there were no design reviews, or assumptions about user behavior turned out to be wrong.

The room specifically calls out how AI assistants are making this worse: developers often assume that a model's output is safe or correct by default, and when an AI system can generate queries, write code, or classify users without limits, that unchecked authority becomes part of the design flaw itself.

**Common issues in 2025:**
- Weak business logic controls (e.g. account recovery or approval flows)
- AI components given unchecked authority or access
- Missing guardrails for LLMs and automation agents
- Debug/test bypasses accidentally left in production
- No consistent review process for abuse cases

**Designing securely** means treating every model as untrusted until proven otherwise, validating all model inputs and outputs, separating system prompts from user-supplied content, requiring human review for high-risk AI actions, and building threat modeling into every stage of development — not just the start.

---

## Room 3: OWASP Top 10 — Insecure Data Handling

The theme here is data itself — what happens when it isn't protected, isn't validated, or isn't verified at the point where it matters most.

### A04: Cryptographic Failures

This category covers data that isn't adequately protected due to missing encryption, faulty implementation, or generally insufficient security measures. Common examples include storing passwords without hashing, relying on outdated or weak algorithms — like **MD5** for hashing or **DES** for encryption, both of which are considered broken by modern standards — exposing encryption keys, or failing to secure data in transit.

**Prevention** means choosing strong, properly implemented encryption, and hashing passwords with slow, purpose-built algorithms like **bcrypt**, **scrypt**, or **argon2** rather than fast general-purpose hash functions. It also means never rolling your own cryptography — relying on trusted, industry-standard libraries instead — and never embedding credentials directly in code, using a proper key management system instead.

### A05: Injection

Injection happens when an application takes user input and, instead of processing it safely, passes it directly to a system that can execute it — a database, a shell, a template engine, or an API.

**Examples covered:**
- SQL injection
- Command injection
- AI prompt injection
- SSTI (Server-Side Template Injection)

**Prevention** starts from treating all user input as untrusted by default: using prepared statements and parameterized queries for SQL, avoiding functions that pass input directly to the system shell for OS commands, and validating input on every form.

### A08: Software or Data Integrity Failures

This category covers situations where an application relies on code, updates, or data without verifying their authenticity, integrity, or origin. That includes software updates, loaded scripts, configuration files, or JSON data — none of which should be trusted just because they arrived from an expected source.

**Prevention** comes down to establishing clear trust boundaries and never assuming code or data is legitimate by default. A common concrete method is using checksums — cryptographic checks that confirm a file hasn't been altered since it was verified.

---

## Closing the Chapter

These three rooms wrap up Cyber Security 101. Where the earlier chapters in this path covered individual tools and platform fundamentals, this one is more about the shape of web application vulnerabilities in general — recognizing that almost every OWASP category boils down to one of a small number of root causes: the server trusting the client too much, an environment set up carelessly, or data moving through the system without being validated or protected at the right point.
