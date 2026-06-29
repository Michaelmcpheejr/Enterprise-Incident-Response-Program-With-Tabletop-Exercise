# Enterprise Incident Response Program - Phase 2
## Detection & Triage Procedures

**Role:** Detection & Triage Lead (and supporting investigation team)  
**Primary Standard:** NIST SP 800-61 Rev. 2 (Analysis phase)  
**Goal:** Determine what happened, scope of impact, evidence for investigation  
**Timeline:** CRITICAL incidents 1-4 hours; HIGH incidents 4-24 hours; MEDIUM incidents 24-48 hours  
**Output:** Investigation summary for Incident Commander and Forensics Lead

---

## Overview: The Triage Process

Triage answers four critical questions:

1. **Is this a real incident** or a false positive?
2. **What is the scope** (which systems, which data)?
3. **What is the impact** (was CUI accessed/exfiltrated)?
4. **What is the timeline** (when did it start, how long was it active)?

This phase is investigation-focused: gather evidence, analyze findings, document results. Phase 3 and 4 handle response and recovery.

---

## Part 1: Initial Investigation (First 1-2 Hours)

### 1.1 Confirm Incident Reality

**Not every alert is a real incident.** First task is to confirm the security event actually occurred and poses a threat.

**Check for false positives:**
- Is this a known, accepted activity? (e.g., system patching, automated scans)
- Does the alert match the system's normal behavior pattern?
- Are there contradictory indicators suggesting the alert is erroneous?
- Has this alert been triggered before (indicating a tuning issue)?

**Gather initial evidence:**
- Pull the raw alert or log entry that triggered the incident
- Photograph the screen showing the alert (timestamp, details)
- Gather related logs from the same timeframe (even if not yet analyzed)
- Document the detection method (SIEM alert, antivirus, user report, etc.)

**Decision point: Is this a false positive?**
- YES: Document as false positive, update alert tuning, close ticket
- NO (or uncertain): Proceed to severity confirmation

---

### 1.2 Confirm Severity Classification

**Re-validate initial severity classification** made during Phase 1. Initial classification may have been quick; now confirm based on emerging evidence.

**CRITICAL indicators to confirm:**
- Active data movement out of network? (Check NetFlow, proxy logs, DLP alerts)
- Ransomware file creation confirmed? (Check file system, backup logs)
- Unauthorized access to systems with CUI? (Check access logs, system logs)
- If YES to any: Confirm CRITICAL classification, proceed to containment coordination

**HIGH indicators to confirm:**
- Malware present on critical system? (Check process execution, file system)
- Unauthorized account or privilege escalation? (Check account logs, privilege audit)
- Evidence of data access by unauthorized entity? (Check file access logs)
- If YES: Confirm HIGH classification, coordinate with IC for investigation scope

**If classification is uncertain,** escalate to Incident Commander immediately rather than guess. IC and CISO can make call based on business context.

---

### 1.3 Preserve Initial System State

**Before investigating further, preserve evidence** from the affected system(s).

**For suspected compromised systems:**
- Capture network connections (netstat -ano, ss -antp on Linux)
- Capture running processes (tasklist /v on Windows, ps aux on Linux)
- Capture memory dump if feasible (without powering down system)
- Document file system state: suspicious files, timestamps, permissions
- Do NOT reboot system (rebooting destroys volatile evidence like memory)

**For data theft/exfiltration:**
- Pull network logs (NetFlow, firewall logs, proxy logs) for affected timeframe
- Identify outbound connections: destination IP, port, volume of data
- Check for suspicious files created or modified
- Identify which user accounts were active during suspected exfiltration

**For malware/ransomware:**
- Isolate system from network (but don't shut down yet)
- Photograph running processes and network connections
- Note file creation timestamps and locations
- Check for lateral movement (evidence of attacker moving to other systems)

**Documentation:**
- Screenshot everything you capture
- Note timestamp of evidence collection
- Document chain of custody (who accessed the system, when, for what purpose)

---

## Part 2: Systematic Investigation Procedure

### 2.1 The TRIAGE Investigation Framework

Use this structured approach to investigate systematically:

**T: Timeline**  
R: Reach (Scope)  
I: Impact Assessment  
A: Attacker/Attack Analysis  
G: Gather Evidence  
E: Escalation/Executive Summary  

---

### 2.2 T - Timeline: When Did This Happen?

**Establish the incident timeline** by identifying key events.

**What you're looking for:**
- First indicator of compromise (earliest log entry showing attack)
- Initial access point (how did attacker get in?)
- Major milestones (lateral movement, privilege escalation, data access)
- Last indicator of activity (when did attacker stop or disappear?)

**Evidence sources for timeline:**

| Evidence Source | What it tells you |
|-----------------|-------------------|
| Firewall/proxy logs | When connection occurred, from where |
| Authentication logs | When accounts were accessed, from where |
| File access logs | When files were accessed or modified |
| Process execution logs | When processes ran, by which user |
| Network logs (NetFlow) | When data left the network, how much |
| System event logs | When system events occurred (logon, privilege change) |
| Endpoint logs | When malware executed, persistence mechanisms |
| Email logs | When suspicious email was delivered (if phishing) |

**Build the timeline:**

```
2:30 PM EST - Phishing email delivered to user@company.com (email log)
2:35 PM EST - User clicked link, credentials captured (proxy log shows odd redirect)
2:40 PM EST - Account user@company.com logged in from unusual IP 203.x.x.x (auth log)
2:45 PM EST - Attacker accessed file share, downloaded CUI files (file access log)
3:10 PM EST - Network exfiltration detected: 2.3 GB data leaving network (NetFlow)
3:15 PM EST - Attacker terminated connection (firewall log)
```

**Timeline output:**
- Earliest indicator: When did compromise begin?
- Duration: How long was attacker active?
- Activity pattern: What did they do while in system?

---

### 2.3 R - Reach (Scope): What Systems Are Affected?

**Determine the scope** of which systems are compromised.

**Scope creep is common:** If one server was compromised, check if attacker moved laterally to others.

**Investigation questions:**
- Which specific systems show evidence of compromise?
- Which accounts were used or compromised?
- Which data repositories were accessed?
- What's the extent of lateral movement?

**Scope determination process:**

**Step 1: Map the attack path**
- Start with the initially compromised system
- Look for evidence of lateral movement (remote sessions, PSExec execution, WMI calls)
- Identify secondary systems the attacker accessed
- Map the path: Entry system → Intermediate systems → Target systems

**Step 2: Check for persistence mechanisms**
- Are there backdoors or persistence tools installed?
- Were administrator accounts created for continued access?
- Are there scheduled tasks or startup scripts that persist access?
- Were SSH keys or authorized_keys modified?
- If persistence found: Attacker intended to maintain access beyond initial intrusion

**Step 3: Validate scope with evidence**
- For each system in scope: Document evidence of compromise (logs, artifacts, file modifications)
- For each system you think might be compromised: Verify with evidence (don't assume)
- Rule out false positives (systems that show alerts but aren't actually compromised)

**Scope output:**
- List of confirmed compromised systems
- List of systems accessed by attacker (may include systems accessed but not compromised)
- Evidence supporting each system in the scope
- List of user accounts compromised

**Example scope statement:**
"Compromised systems: web server 1, file server, domain controller. Attacker accessed but did not compromise: backup server. Compromised accounts: user@company.com, admin-account. Lateral movement confirmed from initial web server to file server."

---

### 2.4 I - Impact Assessment: What Data Is at Risk?

**Determine what data the attacker accessed or exfiltrated.**

This is critical for federal contractors because CUI data triggers regulatory obligations.

**For each compromised system, determine:**
- What data resides on this system? (CUI? PII? Trade secrets?)
- Did the attacker access the data? (check access logs, file modification timestamps)
- Did the attacker exfiltrate the data? (check network logs for bulk data transfer)
- How much data was accessed/exfiltrated? (number of files, data size)
- Who is affected? (employees, customers, government?)

**Data classification questions:**
- Is CUI present on this system? If yes, how much and what type?
- Is personally identifiable information (PII) present? If yes, how many individuals?
- Is intellectual property or trade secrets present?
- Is customer data present?

**Exfiltration confirmation:**
- Pull network logs for suspicious outbound connections during timeframe
- Look for bulk data transfer (unusual volume going to external IP)
- Check proxy logs for file transfers or cloud uploads
- If data was zipped/encrypted before transfer: Indicates intentional exfiltration
- If data was NOT exfiltrated: Impact is significantly lower (breach vs. intrusion)

**Impact statement examples:**

HIGH IMPACT:
"Attacker accessed and exfiltrated CUI files from file server. Confirmed 1,247 CUI documents (total ~850 MB) were transferred to external IP. Exfiltration occurred 15:10-15:15 on 06-15-2026. Data includes contract details and technical specifications."

MEDIUM IMPACT:
"Attacker accessed employee directory on domain controller but no evidence of exfiltration. PII of 237 employees (names, email, phone) was exposed during access but not downloaded. No CUI exposure."

LOW IMPACT:
"Attacker accessed web server but did not access sensitive data. Web server hosts public content only. No CUI or PII at risk."

---

### 2.5 A - Attacker/Attack Analysis: How and Why?

**Determine the attack method and attacker motivation** (if possible).

**Attack vector (how did attacker get in?):**
- Phishing email with malicious attachment
- Phishing email with credential harvesting link
- Exploited known vulnerability on public-facing system
- Compromised third-party account (supplier, partner)
- Insider threat (employee or contractor)
- Brute force attack on remote access service
- Supply chain compromise (malware in software update)

**Attacker motivation (why?):**
- Financial (ransomware, data theft for sale)
- Espionage (nation-state targeting trade secrets or technical data)
- Disruption (sabotage, denial of service)
- Hacktivist (ideological motivation)
- Opportunistic (random targeting, mass exploitation)

**Evidence for motivation:**
- Ransom note: Indicates ransomware attack for financial gain
- Data exfiltration: Indicates theft for financial sale or espionage
- Destructive actions: Indicates sabotage or disruption
- Targeting of specific data: Indicates focused espionage
- No data access: Indicates denial of service or testing

**Analysis output:**
- Confirmed attack vector
- Likely attacker motivation
- Attacker sophistication level (script kiddie, organized cybercriminal, nation-state)
- Whether this appears to be targeted vs. opportunistic

---

### 2.6 G - Gather Evidence: Document Everything

**Evidence standard for federal contractors:**

Evidence must be suitable for:
- Incident investigation (understanding what happened)
- Regulatory notification (what data was exposed?)
- Potential litigation (if lawsuit ensues)
- Law enforcement (if you involve FBI, evidence standards matter)

**Chain of custody for all evidence:**
- Document WHO collected the evidence (name, role)
- Document WHEN it was collected (date, time, timezone)
- Document HOW it was collected (tool, procedure, settings)
- Document WHO has had access (preserve chain of custody)
- Document WHERE it's stored (secure location, access controlled)
- Hash files for integrity (prove they haven't been altered): SHA-256 hash of log files, images, etc.

**Evidence collection standards:**

For log files:
- Export complete logs for relevant timeframe (don't extract summaries)
- Include raw data, not interpretations
- Hash the exported files
- Store in write-protected location

For system artifacts:
- Screenshot suspicious artifacts (running processes, network connections, registry entries)
- Document exact location and content
- Note file hashes if available (prove tampering detection)

For network data:
- Export full packet captures if volume is manageable (not just summaries)
- Export firewall/proxy logs in raw format
- Export NetFlow data for relevant timeframe

**Red flags for poor evidence handling:**
- Altered timestamps (indicates evidence tampering)
- Incomplete logs (missing data for time period in question)
- No chain of custody documentation
- Evidence handled by too many people (contamination risk)
- Evidence not hashed or integrity-checked

---

## Part 3: Completing the Investigation

### 3.1 Root Cause Analysis (High Level)

**Root cause answers: Why was this system compromised?**

This is different from "how did attacker get in" (attack vector). Root cause is the underlying vulnerability or control failure.

**Examples:**

ATTACK: Phishing email with credential harvester  
ROOT CAUSE: Employees not trained to identify phishing; no email authentication (SPF/DKIM) in place  

ATTACK: Exploited unpatched vulnerability on web server  
ROOT CAUSE: Patch management procedures not enforced; vulnerability missed in monthly patching cycle  

ATTACK: Compromised third-party account with system access  
ROOT CAUSE: Vendor credentials stored in shared password file; no MFA on vendor account  

ATTACK: Insider access to CUI without authorization  
ROOT CAUSE: File access controls not enforced; user had broad permissions beyond job role  

**Root cause implications for Phase 4:**
- If root cause is policy/procedure failure: Fix procedures to prevent recurrence
- If root cause is technical vulnerability: Patch or implement control
- If root cause is resource limitation: May require budget allocation to fix

---

### 3.2 Investigation Completion Checklist

Before concluding investigation, confirm:

- [ ] Timeline established (earliest indicator, duration, last activity)
- [ ] Scope documented (all compromised systems, affected data)
- [ ] Impact assessed (CUI? PII? Exfiltration confirmed?)
- [ ] Attack vector identified (how attacker got in)
- [ ] Root cause identified (why they succeeded)
- [ ] Evidence collected and chain of custody documented
- [ ] All findings validated (not assumptions, supported by evidence)
- [ ] Investigation summary drafted
- [ ] Forensics Lead briefed (ready for deep forensic analysis if needed)

---

## Part 4: Investigation Summary & Escalation

### 4.1 Investigation Summary Report

Document findings in a clear, executive-ready format.

**Report structure:**

```
INCIDENT INVESTIGATION SUMMARY

Incident ID: [INC-2026-06-015]
Date: [06-15-2026]
Severity: [HIGH]
Status: [Investigation Complete - Awaiting Containment/Recovery Decision]

TIMELINE:
- 2:30 PM EST: Phishing email delivered
- 2:35 PM EST: User clicked malicious link
- 2:40 PM EST: Credentials compromised, account logged in from external IP
- 2:45 PM EST-3:15 PM EST: Attacker accessed file server, exfiltrated CUI
- 3:15 PM EST: Attacker disconnected

SCOPE:
- Compromised systems: File server, domain controller
- Affected accounts: user@company.com, temp-admin account
- Systems accessed but not compromised: Backup server (no persistent access)

IMPACT:
- CUI data exfiltrated: 1,247 documents, ~850 MB
- Data includes: Contract specifications, technical drawings, customer lists
- PII exposure: None confirmed
- Business impact: Significant (trade secrets and customer confidentiality compromised)

ATTACK VECTOR:
- Phishing email with credential harvesting link
- User entered credentials on fake login page
- Attacker used compromised credentials to access file server
- Lateral movement: File server to domain controller

ROOT CAUSE:
- Inadequate phishing training (employee couldn't identify fake login page)
- No MFA on file server access (password compromise sufficient for entry)
- No email authentication (phishing email appeared legitimate)

EVIDENCE:
- Email logs: Phishing email (hash: xxx)
- Proxy logs: Credential entry on fake site (IP xxx)
- Authentication logs: Login from external IP (time-stamped)
- Network logs: 850 MB exfiltration (NetFlow data, packet captures)
- File access logs: CUI file access timestamps

FORENSICS RECOMMENDATION:
- Full disk image of file server for deep analysis
- Memory analysis of domain controller (check for persistence)
- Detailed timeline analysis of attacker activities

NEXT STEPS:
- Containment: Isolate compromised systems, reset compromised accounts
- Investigation: Forensics team deep analysis (separate from this document)
- Recovery: Rebuild systems from known-good backups
- Notification: Prepare breach notification for affected parties
```

---

### 4.2 Escalation to Incident Commander

**Brief Incident Commander with:**
- Severity confirmation (CRITICAL/HIGH/MEDIUM/LOW)
- Scope (what's compromised)
- Impact (what data is at risk)
- Next steps (containment/recovery actions needed)
- Timeline for action (urgent vs. can wait)

**For CRITICAL severity:**
- Brief call with IC, CISO, CFO
- Recommendation: Immediate system isolation or data preservation action
- Alert to Board Chair if CUI exfiltration confirmed

**For HIGH severity:**
- Email to IC with summary
- Recommend next steps based on investigation findings
- Await IC decision on containment/recovery action

---

### 4.3 Handoff to Forensics Lead & Phase 3

**Forensics Lead takes investigation deeper if needed:**
- Deep memory analysis (discover malware or attacker tools)
- Detailed artifact analysis (registry, file system, application logs)
- Timeline reconstruction (precise sequence of events)
- Attribution analysis (who was the attacker?)

**Investigation summary provides Forensics Lead with:**
- Focus areas (which systems to analyze)
- Context (known timeline, known scope, known impact)
- Key questions (did persistence exist? Was this targeted?)

---

## Part 5: Documentation & Retention

### 5.1 Investigation Records to Preserve

For each incident, retain:
- Investigation summary report
- All evidence collected (logs, screenshots, memory dumps, packet captures)
- Chain of custody documentation
- Investigation timeline (with supporting evidence)
- Communications about the incident (emails, chat logs, decision documentation)
- Forensics analysis results (if conducted)
- Breach notification determination (was breach notification required?)

**Retention:** Minimum 7 years (federal requirement for incident records)


