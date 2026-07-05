### Introduction to SOC Alerts

This room picks up where the general SOC overview left off, going deeper into what actually lands on an analyst's screen: the alert itself. It covers where alerts come from, how they're structured, how L1 analysts decide which one to work on first, and what happens when an alert turns out to be real.

---

#### From Event to Alert

An alert doesn't appear out of nowhere — it's the end of a pipeline:

1. **Event occurs** — a login, a process launch, a file download.
2. **Logging** — the OS, firewall, or cloud provider records it.
3. **Log shipping** — logs are forwarded to a SIEM or EDR.
4. **Alert generation** — a detection rule fires when a specific pattern matches.

The reason this pipeline matters: a SOC can take in millions of log lines a day. Without the detection layer turning that into a handful of alerts, no human team could keep up. The alert is essentially a pre-filtered signal — but it's still just a _hint_, not a verdict, which is why triage exists as a separate step.

#### Where Alerts Live

Different platforms handle alerts differently, and picking the right one matters for how a SOC scales:

| Platform  | Examples                 | Why you'd use it                                                                                                                                          |
| --------- | ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SIEM      | Splunk ES, Elastic SIEM  | Strongest native alert management — the default choice for most SOC teams                                                                                 |
| EDR / NDR | MS Defender, CrowdStrike | Good for endpoint/network visibility, but alert handling is usually still funneled into a SIEM or SOAR rather than worked from the EDR dashboard directly |
| SOAR      | Splunk SOAR, Cortex SOAR | Used by larger teams to aggregate alerts across multiple tools into one queue                                                                             |
| ITSM      | Jira, TheHive            | Some teams manage alert tickets through general-purpose or adapted ticketing tools (this pairing is straight from how the room framed it)                 |

#### Who Touches an Alert

- **L1 Analyst** — first responder. Reviews the alert, decides real vs. noise, escalates if it's real.
- **L2 Analyst** — takes escalated alerts, does deeper investigation and remediation.
- **SOC Engineer** — makes sure alerts actually contain enough context to be triaged efficiently in the first place.
- **SOC Manager** — tracks speed and quality of triage so nothing real slips through.

A typical L1 handles anywhere from zero to a hundred alerts a day, which is exactly why prioritization (below) matters as much as the investigation itself.

---

#### Anatomy of an Alert

Every alert carries the same core fields, regardless of platform:

| Field       | What it tells you                                                                     |
| ----------- | ------------------------------------------------------------------------------------- |
| Alert Time  | When the alert fired (usually a few minutes _after_ the actual event)                 |
| Alert Name  | Summary based on the detection rule, e.g. "Unusual Login Location"                    |
| Severity    | Low → Medium → High → Critical, set by detection engineers but adjustable by analysts |
| Status      | New, In Progress, or Closed                                                           |
| Verdict     | True Positive (real) or False Positive (noise)                                        |
| Assignee    | Analyst currently owning the alert                                                    |
| Description | The rule's logic, why it might indicate an attack, and sometimes how to triage it     |
| Fields      | The specific values that tripped the rule — hostname, command line, etc.              |

#### Picking the Next Alert

With a queue full of alerts, the common approach is a three-step filter:

1. **Filter** — skip anything already reviewed or being worked by a teammate; focus only on new, unresolved alerts.
2. **Sort by severity** — Critical first, since detection rules are tuned so critical alerts are more likely to be real, high-impact threats.
3. **Sort by time** — oldest first, on the logic that an attacker from an older alert has likely progressed further (e.g. already exfiltrating data) than one who's just started.

---

#### Investigating and Reporting

Investigation is the most technical step — pulling logs from the SIEM/EDR to establish whether the activity is legitimate. The recommended approach is to identify the **target** (user, host, cloud resource, network, site) and the **action** (suspicious login, malware, phishing), then check the surrounding events for anything else suspicious in that window.

Once a verdict is reached, it gets written up using the **Five Ws**:

- **Who** triggered it
- **What** exactly happened
- **When** it started and ended
- **Where** it happened (device, IP, site)
- **Why** — the reasoning behind the verdict, considered the most important part of the report

**Worked example from the room:** an email spoofing "Microsoft Support" landed in an IT Manager's inbox warning of a 600% Teams pricing increase, pushing urgency and a `REPORT.rar` attachment. It failed both SPF and DKIM. On its own, a failed SPF/DKIM check is common enough with legitimate misconfigured senders — the detail that actually tips it into **True Positive** territory is the combination: spoofed sender + failed authentication + urgency language + an executable-adjacent archive format (`.rar`) standing in for what should have been a plain PDF report. That's the kind of reasoning a Five Ws writeup is meant to capture — not just "it's phishing" but _why_ the evidence adds up to that conclusion.

#### Escalation

An alert goes to L2 when: it looks like a major attack, remediation is needed (isolating a host, resetting credentials), external parties need to be looped in (customers, law enforcement), or the L1 analyst genuinely isn't sure. Escalating an alert doesn't mean starting over — the whole point of the L1 report is that L2 can pick up from the documented context instead of re-investigating from scratch.

A few procedural notes that stood out:

- If a user's account is compromised, you verify the login _with the user_, but never over the same chat platform that might be compromised — a call or alternate channel is used instead.
- If L2 doesn't respond during a critical alert, the escalation path goes L2 → L3 → manager, not "wait it out."
- Realizing after the fact that an alert was misclassified isn't something to sit on — it gets reported to L2 immediately, since an attacker can stay quiet for weeks before doing damage.

---

#### Measuring the SOC

| Metric                      | Formula                          | What it's really measuring                                                                                   |
| --------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Alerts Count (AC)           | Total alerts received            | Analyst workload — 5–30/day per analyst is considered healthy; too low can mean blind spots, not a quiet day |
| False Positive Rate (FPR)   | False Positives ÷ Total Alerts   | Noise level — above ~80% and analysts start tuning alerts out entirely                                       |
| Alert Escalation Rate (AER) | Escalated ÷ Total Alerts         | L1 maturity — target is under 50%, ideally under 20%                                                         |
| Threat Detection Rate (TDR) | Detected Threats ÷ Total Threats | Should always be 100% — a miss here isn't a bad metric, it's a breach                                        |

On top of alert-level metrics, teams also track response time against SLA targets: detecting an attack within ~5 minutes (MTTD), acknowledging a new alert within ~10 minutes (MTTA), and actually containing it within ~60 minutes (MTTR). These numbers only matter together — a fast MTTD with a slow MTTR just means you found the fire quickly and still let it spread.

---

#### Hands-On: SOC Scenario Simulator

The room's practical component puts you in a SOC seat across three escalating scenarios — an unhappy customer after a slow breach response, a delayed alert, and a burned-out analyst team — and asks you to match each one to its root metric, the fix, and who should own that fix.

Completed all three scenarios and captured the flags.

> **Personal Note:** I didn't know "contact via a potentially breached chat channel" was even a real risk to think about — makes sense once it's pointed out (if the attacker still has access, you're literally notifying them that you're onto them), but it's not something I would have thought of on my own. I also noticed the whole triage flow in this lab reads like a fixed SOP rather than something with much room for individual judgment calls — the escalation path, the communication rules, the prioritization order are all pretty prescriptive, which tracks with what SOC work seems to actually be: consistency and speed matter more than creativity.
