# Phishing Analysis

---

## Overview

Phishing Analysis covers how phishing attacks work, how to investigate a suspicious email as an analyst, and the technical mechanisms organizations use to prevent phishing from reaching end users in the first place. The chapter moves from email fundamentals (how email is actually delivered), into the anatomy and techniques of phishing attacks, then into the tools used to analyze a suspicious email, and finally into the authentication standards (SPF, DKIM, DMARC, S/MIME) and organizational defenses that reduce phishing risk at scale.

---

## Email Fundamentals

### Anatomy of an Email Address

| Component   | Description                                                                          |
| ----------- | ------------------------------------------------------------------------------------ |
| Username    | The user mailbox that identifies the specific recipient on the mail server           |
| `@` symbol  | Separates the username from the domain and tells the system where to route the email |
| Domain name | Specifies the mail server responsible for receiving the message                      |

Example: `user@domain.com`

### Email Delivery Protocols

Several protocols work together behind the scenes to move a message from sender to recipient, each with a specific role.

| Protocol | Full Name                        | Role                                                                                                                                            |
| -------- | -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| SMTP     | Simple Mail Transfer Protocol    | Sends emails                                                                                                                                    |
| POP3     | Post Office Protocol             | Downloads emails to a single device. Messages are typically removed from the server after download and are only accessible from that one device |
| IMAP     | Internet Message Access Protocol | Syncs emails across multiple devices. Messages stay on the server unless explicitly deleted, and sent messages are also stored server side      |

### The Email Journey

1. The sender's email client sends the message to their mail server using SMTP.
2. The sending mail server queries DNS for the recipient domain's mail server.
3. DNS returns the address of the recipient's mail server.
4. The message is delivered across the internet to the recipient's server.
5. The recipient's email client connects to their mail server to check the mailbox.
6. The message is downloaded (POP3) or synced (IMAP) to the recipient's device.

Seeing this full journey laid out made it clear why header analysis later in the chapter is so useful. Every hop in that journey leaves a trace in the email header, which is exactly what an analyst pulls apart during an investigation.

---

## Types of Phishing

| Type           | Description                                                                                                                      |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Spam           | Unsolicited bulk email sent to a large number of recipients. A more malicious form of spam is often called malspam               |
| Phishing       | Emails that impersonate a trusted entity to trick recipients into revealing sensitive information                                |
| Spear Phishing | A targeted form of phishing aimed at a specific individual or organization, often using personalized information                 |
| Whaling        | A form of spear phishing that specifically targets high level executives (CEO, CFO) to obtain sensitive data or financial access |
| Smishing       | Phishing conducted via SMS or text message, targeting mobile users                                                               |
| Vishing        | Phishing carried out through voice calls, using social engineering over the phone                                                |

---

## Anatomy of a Phishing Email

- **Spoofed From Address**: The sender's email is spoofed to appear as a trusted entity, for example `noreply@microsof.com`.
- **Urgent Subject or Message**: The email manufactures urgency, for example "Your account will be locked in 24 hours."
- **Brand Impersonation**: The email mimics a legitimate organization, matching logos or colors.
- **Grammar and Spelling Issues**: Errors may appear, though these are becoming less common now that attackers use AI to write more convincing messages.
- **Generic Content**: The message lacks personalization, for example "Dear Customer" instead of a name.
- **Hidden or Shortened Links**: Hyperlinks disguise their true destination, for example `bit.ly/secure-login`.
- **Malicious Attachments**: Files are disguised as legitimate documents, for example `invoice.pdf.exe`.

### Defanging

Defanging makes URLs, domains, or email addresses unclickable so they can be shared or documented without risking an accidental click. It works by replacing special characters, such as `@` in an email or `.` in a URL, with alternate characters.

- Original URL: `http://www.suspiciousdomain.com`
- Defanged URL: `hxxp[://]www[.]suspiciousdomain[.]com`

---

## Common Phishing Techniques

- **Spoofed email address**: Mimicking a trusted service to gain immediate credibility.
- **URL shortening**: Using redirection services to hide the true destination of a link.
- **Branded HTML**: Impersonating legitimate corporate imagery to create a sense of authenticity.
- **Pixel tracking**: Embedding invisible images that notify the sender when the email is opened.
- **Link manipulation**: Masking a malicious destination behind something like a fraudulent tracking number.
- **Artificial urgency**: Creating a narrow window for action to pressure a quick response.
- **Brand impersonation**: Layering trusted brands, such as Microsoft or Adobe, to build false confidence.
- **Link redirection**: Chaining URLs together to hide the final malicious destination from basic email filters.
- **Credential harvesting**: Deploying a fake login portal to capture and exfiltrate usernames and passwords.

### Case Example: A Fake Netflix Billing Email

This example combined several techniques at once:

- The sender's display name was set to something like "Netllx Billing" to appear legitimate at a glance.
- A suspended account notice was used to create urgency and pressure quick action.
- Noticeable misspellings of "Netflix" were present in the display name and body.
- A file attachment was used instead of a direct link, which hides the malicious payload from link-scanning filters that only check URLs.

### Case Example: First Observations on a PayPal Themed Email

- The subject line referenced a fake transaction to trigger a hasty reaction.
- The From address showed `service@paypal.com` as the display name, but the actual sending address was something unrelated like `gibberish@sultanbogor.com`, an immediate red flag.
- The To address was an unusual recipient address, not a normal personal inbox address, suggesting the message was part of a bulk or scraped list.
- The email body itself was visually well designed, which is a reminder that visual polish is not proof of legitimacy. The button's underlying link (its `href`) is what needs checking, not how the button looks. A tool like WhereGoes is useful here, especially when the link has been shortened.

---

## Phishing Analysis Tools

### Header Artifacts

| Artifact                | What It Tells You                                                              |
| ----------------------- | ------------------------------------------------------------------------------ |
| Sender Email Address    | Identifies the source location where the email originated                      |
| Sender IP Address       | The specific source IP, useful for a reverse lookup to find the origin network |
| Email Subject Line      | Often reveals the psychological angle of the attack, such as forced urgency    |
| Recipient Email Address | Tracks who the message was targeted at, via the To, CC, or BCC fields          |
| Reply-To Email Address  | The address where any reply from the victim will actually be sent              |
| Date and Time           | The exact moment the email was transmitted                                     |

Basic details like sender and recipient are visible in normal clients like Gmail or Yahoo, but deeper artifacts such as the sender's real IP address and Reply-To data require pulling the email's raw source code.

**Tools**: Messageheader (part of the Google Admin Toolbox) and Message Header Analyzer both extract the sender's IP address and routing path, and highlight misconfigurations from pasted header text.

### IP and URL Reputation Analysis

Checking the reputation of an identified IP or URL is essential for determining whether the underlying infrastructure is tied to known malicious activity.

- **IPinfo**: Gathers details about an IP address, including geographic location and the associated organization, which helps determine if the IP looks malicious.
- **URLScan.io**: Safely investigates a website by simulating a real browsing session, recording page activity, capturing a screenshot, and revealing the site's underlying behavior without direct exposure.
- **Talos IP & Domain Reputation Center**: A Cisco threat intelligence tool that assesses the reputation of IPs, domains, and networks, and shows whether an indicator has been linked to malicious activity.

### Email Body Analysis

- **Determining link destinations safely**: Right-click a link and select Copy Link Address (exact wording varies by browser or email provider) to inspect the URL without visiting it.
- **URL extraction tools**: Automate the discovery of links by parsing raw email content, which saves time and reduces the risk of missing a hidden or obfuscated URL. CyberChef can perform this extraction during email analysis, which lines up with how it is already used elsewhere in this portfolio for data transformation tasks.

### Email Attachments

Attachments should never be downloaded to a primary machine. They should only be downloaded inside a controlled environment such as a virtual machine or sandbox, to prevent accidental execution.

Once safely downloaded in a Linux environment, a tool like `sha256sum` generates a unique hash of the file for reputation checks:

```bash
sha256sum shady_attachment.pdf
```

This outputs the file's SHA256 hash followed by the file name. That hash can then be checked against:

- **Talos IP & Domain Reputation Center**, which can return labels such as Phishing, Malicious, or Spam for a known bad hash.
- **VirusTotal**, a widely used aggregate scanner that checks files, URLs, IPs, and domains against dozens of security vendors, and returns detailed detection results.

The reasoning behind this workflow makes sense once you see it laid out: you never want to risk detonating a suspicious file on a real machine, so you isolate it, fingerprint it with a hash, and then let reputation databases tell you what it is before touching it any further.

### Malware Sandboxes

| Sandbox         | Notes                                                                                                                                                                 |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ANY.RUN         | Interactive sandbox that lets analysts directly interact with the environment in real time, monitoring processes, network activity, and system changes as they happen |
| Hybrid Analysis | A free sandbox that delivers detailed insights into file behavior, system changes, network activity, and IOCs                                                         |
| JOESandbox      | Built by JOESecurity, performs both static and dynamic analysis and generates comprehensive reports on behavior, IOCs, and threat classification                      |

The main advantage across all three is that an analyst does not need advanced malware reverse engineering skills to understand what a malicious attachment does. Uploading the file and observing what URLs it contacts, what it downloads, and what it changes on the system is often enough to build a picture of intent.

### PhishTool

PhishTool is a platform built to streamline phishing investigations for SOC analysts triaging user-reported emails, threat intelligence analysts collecting indicators, and researchers investigating phishing kits. It combines threat intelligence, OSINT, email metadata, and automated analysis workflows into a single interface, and automates much of the manual work involved in analyzing a suspicious email.

**Identifying artifacts**: Upon uploading a file, PhishTool immediately displays key email artifacts. It offers three ways to view the message: a rendered HTML view similar to a normal inbox, the raw HTML code, and the full message source.

**Further analysis**: Top navigation tabs let an analyst move through authentication results, transmission paths, and embedded URLs, with attachments reviewable directly inside the platform. PhishTool also integrates with VirusTotal, so reputation and detection results appear without leaving the tool.

**Resolving the case**: Once analysis is complete, findings can be formally documented. The email can be marked as malicious with specific artifacts flagged, such as sender addresses, originating IP, and embedded URLs. Investigation notes are added and the case is marked Resolve, which mirrors how a real SOC closes out a case.

---

## Phishing Prevention Mechanisms

### SPF (Sender Policy Framework)

**Purpose**: Authenticates the sender of an email so ISPs can verify that a mail server is authorized to send email on behalf of a specific domain.

**Structure**: An SPF record is a DNS TXT record containing a list of IP addresses allowed to send email for a domain.

**Workflow**:

1. The receiving mail server checks the sending domain's SPF record.
2. It verifies whether the sending server is authorized to send on behalf of that domain.
3. Delivery of the email is based directly on the result of that check.

**Verification Results and Actions**:

| Result              | Action                                 |
| ------------------- | -------------------------------------- |
| Pass, Neutral, None | Accept (allow and process the email)   |
| SoftFail, PermError | Flag (mark as suspicious but allow)    |
| Fail, TempError     | Reject (immediately discard the email) |

**Tools**: SPF Surveyor (visual look at DNS records), Google Admin Toolbox Messageheader (analyzes delivery details using an email's full header).

### DKIM (DomainKeys Identified Mail)

**Purpose**: An open standard used to authenticate a sent email and establish DMARC alignment.

**Structure**: A DKIM record exists in DNS but has a more complex structure than SPF. Its main advantage is that it can survive email forwarding, which makes it more reliable than SPF in forwarded contexts.

**Workflow**:

1. The sending mail server uses a private key to apply a unique digital signature to the message header.
2. The receiving mail server retrieves the corresponding public key from the sending domain's DKIM record in DNS.
3. The receiving server uses the public key to verify the signature. If it matches, the email is authentic. If not, it may be flagged or rejected.

**Record Components**:

| Tag              | Meaning                                                                                    |
| ---------------- | ------------------------------------------------------------------------------------------ |
| `v=DKIM1`        | Specifies the DKIM version being used (optional)                                           |
| `k=rsa`          | Defines the cryptographic key type, RSA is standard                                        |
| `p=<public_key>` | The public key the receiving server matches against the private key to verify authenticity |

DKIM record formats can vary depending on the mail provider or system.

**Tools**: DKIM Record Checker, DKIM Validator.

### DMARC (Domain-based Message Authentication, Reporting and Conformance)

**Purpose**: An open standard that ties the results of SPF and DKIM to the actual content of an email, using a concept called alignment to verify the sender's domain matches the domains authenticated by SPF and DKIM.

**Policy Enforcement**: If alignment fails, DMARC gives the receiving server explicit instructions on how to handle the message.

**Record Components**:

| Tag                                 | Meaning                                                                                                               |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `v=DMARC1`                          | Identifies the DMARC version (required)                                                                               |
| `p=quarantine`                      | The active policy. Common values are `none` (monitor only), `quarantine` (move to spam), or `reject` (block entirely) |
| `rua=mailto:postmaster@website.com` | Optional reporting address for aggregate data and alignment reports                                                   |

**Case Study**: Testing a domain like `microsoft.com` against a DMARC inspector shows full passes across all alignment checks, including use of the `p=reject` policy, which ensures any email failing DMARC is discarded immediately. Seeing a heavily targeted brand like Microsoft run the strictest possible policy makes it clear why `p=reject` is considered best practice once an organization is confident its legitimate mail sources are fully accounted for.

**Tools**: DMARC Domain Checker, DMARC Record Inspectors (such as those from dmarcian).

### S/MIME (Secure/Multipurpose Internet Mail Extensions)

**Purpose**: A security protocol for transmitting digitally signed and encrypted messages, built on public key cryptography using a protected private key and an openly distributed public key.

**Digital Signature Component**: The sender signs the email with their private key, and the recipient verifies the sender's identity using the sender's public key. This provides:

- **Authentication**: Confirms the sender's identity via a validated digital certificate.
- **Non-repudiation**: The sender cannot deny having sent the message.
- **Data Integrity**: Detects whether the message content was altered after signing.

**Encryption Component**: The sender encrypts the message body using the recipient's public key, so only the recipient's private key can decrypt it. This provides:

- **Confidentiality**: Keeps the message content private and readable only by the intended recipient.

Coming from a WordPress background, this is the section that connected most directly with prior experience. SPF, DKIM, and DMARC records were things previously configured in email plugins (for clients sending transactional or marketing email from their domain) without fully understanding why each one mattered or what happened behind the scenes when a check failed. Seeing the actual verification workflow explained end to end filled in gaps that were previously just "follow the plugin's setup wizard."

---

## How Organizations Stop Phishing

### Technical Defenses

- **Email Filtering**: Evaluates messages using inbound IP and domain reputation, allowing flagged mail to be blocked or quarantined automatically.
- **Secure Email Gateways (SEGs)**: Inline security appliances that scan messages specifically for domain impersonation, spoofing, and complex phishing techniques that standard filters miss.
- **Link Rewriting**: Replaces unknown or suspicious URLs in incoming email with secure redirected links, giving backend systems time to scan and verify the true destination before a user can click through.
- **Sandboxing**: Automatically isolates and executes suspicious links or attachments in a virtualized environment to observe malicious behavior before it reaches the user.

### User Facing Tools and Training

- **Trust and Warning Indicators**: Visual banners added by modern email clients, such as "External Sender" or "Suspicious Link" warnings, or trusted badges for verified senders.
- **Phishing Reporting**: One-click reporting built into the email client, letting users instantly forward suspicious messages to the security team.
- **User Awareness Training**: Education modules teaching employees to identify phishing indicators, recognize social engineering tactics, and follow safe handling practices.
- **Phishing Simulation Exercises**: Controlled, benign phishing campaigns run internally to measure and reinforce how well employees spot and report suspicious email.

---

## Personal Notes

> This chapter landed differently than the others because of my background as a WordPress designed and developer. I had configured SPF, DKIM, and DMARC records before for client email plugins during freelance projects, but I never truly understood what was happening behind the scenes as I was only following the documentations on setting up the configurations. This chapter provided me much clarification of the whole email process and the reason to why SPF, DKIM and DMARC are important.
>
> The phishing techniques section also changed how I look at my own inbox now. Things like pixel tracking and link redirection chains were not things I thought about before, and I catch myself checking sender addresses and hovering over links a lot more carefully since going through this material.
>
> Out of everything covered in SOC Level 1 so far, this is the most familiar chapter given my frontend and WordPress background, since email deliverability and authentication were already part of that world. The difference now is understanding the security reasoning behind each mechanism instead of just implementing it because a setup guide said to.
