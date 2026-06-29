# Enterprise Incident Response Program - Phase 1
## IR Framework & Procedures

**Program Type:** Operational framework (federal contractor context)  
**Primary Standard:** NIST SP 800-171 Section 3.6 (Incident Response)  
**Secondary Alignment:** NIST SP 800-61 Rev. 2 (Computer Security Incident Handling Guide)  
**Target Organizations:** Defense subcontractors, federal contractors handling CUI  
**Scope:** Detection through post-incident reporting

---

## Core Principle

Incident response is not optional—it's mandated compliance (NIST 800-171 3.6.1) and business-critical. This framework ensures your organization can detect incidents quickly, respond decisively, preserve evidence for investigation, communicate clearly to leadership, and improve continuously.

---

## Part 1: Incident Response Program Foundation

### 1.1 What an Incident Is (Definition)

**NIST Definition:** An incident is a violation of computer security policies or acceptable use policies or an event that has potential to impact CIA (confidentiality, integrity, availability) of systems or data.

**For federal contractors handling CUI, this includes:**
- Unauthorized access to systems or data
- Malware infection or execution
- Data exfiltration or unauthorized disclosure
- System unavailability (ransomware, DoS)
- Configuration changes (unauthorized or suspect)
- Insider activity (malicious or negligent)
- Supply chain compromise affecting your systems

**Distinction:** Not every security event is an incident. A misconfigured firewall rule is not an incident until it creates a security violation or breach condition.

---

### 1.2 Why IR Matters for Federal Contractors

**Compliance requirement:** NIST 800-171 Section 3.6.1 requires:
- Documented incident handling procedures
- Capability to detect incidents within a reasonable timeframe
- Procedures to respond, investigate, and recover

**Breach notification requirement:** If CUI is exposed:
- Notify affected individuals (if personally identifiable)
- Notify law enforcement and U.S. Cert if government data is involved
- Timeline: As quickly as possible, typically within 30 days

**Board accountability:** Incidents involving CUI trigger:
- Immediate C-suite notification (CEO, CISO, CFO)
- Board reporting (in next board meeting or immediately if critical)
- Insurance notification (cyber liability policy)
- Regulator notification (depending on incident type)

**Business impact:** Unmanaged incidents lead to:
- Compliance violations (audit findings, penalties)
- Loss of customer trust (contracts at risk)
- Operational downtime (revenue impact)
- Reputation damage (public disclosure if breached)

---

## Part 2: IR Program Structure & Roles

### 2.1 Incident Response Team (IRT)

Effective IR requires clearly defined roles with decision authority.

**Incident Commander (IC)**
- Authority: Makes containment and response decisions during incident
- Responsibilities: Leads IR team, communicates status to leadership, coordinates timeline
- Qualifications: CISO, Security Manager, or Senior Security Engineer
- During incident: Single point of escalation (avoids conflicting decisions)

**Detection & Analysis Lead**
- Authority: Determines if an event qualifies as an incident
- Responsibilities: Initial investigation, severity assessment, escalation decision
- Qualifications: SOC Analyst, Security Operations Manager, or equivalent
- Key task: Separates false positives from true incidents (speed matters)

**Containment & Response Lead**
- Authority: Executes containment procedures (isolation, blocking, etc.)
- Responsibilities: Implements playbook actions, prevents spread, preserves evidence
- Qualifications: Systems Administrator, Security Operations, or Network Engineer
- Key task: Stops damage while preserving investigation capability

**Investigation & Forensics Lead**
- Authority: Directs forensic analysis and root cause investigation
- Responsibilities: Collects evidence, preserves chain of custody, determines breach scope
- Qualifications: Forensic analyst, Security investigator, or Senior security engineer
- Key task: Answering "What happened?" and "Why did it happen?"

**Communications Lead**
- Authority: Crafts all incident communications (internal and external)
- Responsibilities: Notifies stakeholders, manages disclosure timeline, coordinates messaging
- Qualifications: Communications, Legal, or Risk/Compliance Manager
- Key task: Ensuring accuracy and legal compliance in all communications

**Executive Sponsor (CISO/CRO)**
- Authority: Approves major decisions (breach notification, public disclosure, etc.)
- Responsibilities: Escalation point for critical decisions, board communication
- Qualifications: CISO, Chief Risk Officer, or Chief Compliance Officer
- Key task: Ensuring business and legal implications are understood

**Optional: Third-Party Support**
- External forensics firm (if internal expertise unavailable)
- Legal counsel (for breach notification, regulatory requirements)
- Insurance broker (cyber liability, notification requirements)

**Note:** In smaller organizations, one person may hold multiple roles. Define clearly to avoid decision delays.

---

### 2.2 Incident Severity Classification

Severity determines response speed and escalation level. Classification must be made during detection phase, before full investigation.

**CRITICAL (Severity 4 / Red)**

**Definition:** Active data exfiltration of CUI, ransomware encryption of critical systems, or other threat that immediately impacts business operations or creates imminent regulatory violation.

**Examples:**
- Ransomware deployed on file servers containing CUI
- Active data exfiltration detected (data leaving network in bulk)
- Unauthorized access to systems controlling CUI with evidence of data access
- System compromise affecting ability to deliver services to customers

**Response timeline:** Detection to Incident Commander notification = within 15 minutes  
**Escalation:** Immediate notification to CISO, CEO, CFO  
**Investigation timeline:** Begin full investigation within 1 hour  
**Board notification:** Immediate (same day)

---

**HIGH (Severity 3 / Orange)**

**Definition:** Confirmed unauthorized access, malware infection on critical system, or data integrity compromise that requires containment within hours to prevent escalation to CRITICAL.

**Examples:**
- Malware detected on endpoint with CUI access
- Unauthorized administrator account created on critical system
- Data integrity issue detected (unauthorized file modifications)
- Phishing campaign with successful credential harvesting

**Response timeline:** Detection to Incident Commander notification = within 1 hour  
**Escalation:** Notification to CISO, direct manager  
**Investigation timeline:** Begin containment procedures within 2 hours  
**Board notification:** Within next business day if data compromise confirmed

---

**MEDIUM (Severity 2 / Yellow)**

**Definition:** Security event with potential impact that requires investigation but does not immediately threaten data or operations. Investigation may determine it's a false positive.

**Examples:**
- Suspicious login from unusual location (not yet confirmed unauthorized)
- Potential vulnerability exploitation (system reachable but not confirmed compromised)
- Elevated privilege usage without clear authorization (may be legitimate business need)
- Network reconnaissance detected but no data access confirmed

**Response timeline:** Detection to investigation team notification = within 4 hours  
**Escalation:** Notification to Security Operations Manager  
**Investigation timeline:** Begin investigation within 8 hours  
**Board notification:** Only if investigation confirms data compromise

---

**LOW (Severity 1 / Green)**

**Definition:** Security event with minimal impact potential or false positive indicator requiring verification.

**Examples:**
- Failed login attempts (normal for large organizations)
- Antivirus quarantine of known malware signature (no evidence of execution)
- Policy violation without security impact (e.g., USB drive used for backup)
- Security alert flagged by tool but inconsistent with other evidence

**Response timeline:** Detection to triage = within 24 hours  
**Escalation:** Triage team determines if it requires further investigation  
**Investigation timeline:** Proceed only if additional evidence suggests actual security event  
**Board notification:** No

---

### 2.3 Initial Escalation Procedures

**Who escalates and when:**

**Detection & Analysis Lead determines severity → Decides escalation level**

**CRITICAL incidents:**
- Immediately call Incident Commander (do not wait for email response)
- Simultaneously notify CISO (phone call)
- Simultaneous notification to CFO (prepare for potential financial impact)
- Log incident into tracking system with severity classification and timestamp

**HIGH incidents:**
- Email notification to Incident Commander within 15 minutes with initial findings
- If IC doesn't respond within 30 minutes, escalate to CISO directly
- Notify CISO of HIGH severity incident within 1 hour

**MEDIUM incidents:**
- Email or ticketing system notification to Incident Commander
- Begin initial investigation (don't wait for IC approval)
- Escalate to CISO only if investigation confirms data compromise

**LOW incidents:**
- Log in ticketing system for tracking
- Triage team investigates; no escalation unless severity increases

**Board escalation trigger (for CRITICAL only):**
- CISO notifies Board Chair or Audit Committee Chair immediately if:
  - CUI data exfiltration confirmed
  - Regulatory notification likely
  - Significant business interruption expected
  - Media exposure possible
- Formal board notification at next scheduled board meeting or emergency board call

---

## Part 3: Initial Response Procedures

### 3.1 Detection Phase (Before IR Engagement)

**How incidents are discovered:**

**Automated:** Security tools detect anomalies (SIEM alerts, antivirus, IDS/IPS, endpoint detection)  
**Manual:** Security team notices suspicious activity during monitoring  
**Notification:** Customer, partner, or law enforcement reports suspected breach  
**Incident Report:** Employee reports suspicious activity  

**Detection is not investigation.** The goal is to quickly identify events that MIGHT be incidents, classify severity, and escalate. Full investigation happens in Phase 2.

---

### 3.2 Initial Response (First 30 Minutes)

**Goal:** Confirm incident is real, establish Incident Commander authority, begin containment planning, preserve evidence.

**Step 1: Acknowledge alert (Within 5 minutes)**
- Detection lead receives alert
- Confirms alert is not false positive (check for known false positive patterns)
- If unsure, escalate (better to over-escalate than miss real incident)

**Step 2: Classify severity (Within 10 minutes)**
- Does the event match CRITICAL criteria? (active exfil, ransomware, active unauthorized access)
- Does it match HIGH criteria? (confirmed access, malware, data modification)
- Does it match MEDIUM criteria? (suspected activity requiring investigation)
- Does it match LOW criteria? (likely false positive or minor event)

**Step 3: Escalate per severity (Within 15 minutes for CRITICAL, 30 minutes for HIGH)**
- Follow escalation procedures above
- Do not wait for perfect information before escalating

**Step 4: Incident Commander assumes control**
- IC confirms initial findings
- IC authorizes containment procedures (if needed)
- IC schedules IRT coordination call within 1 hour for CRITICAL/HIGH

**Step 5: Preserve initial evidence**
- If possible, capture initial system state before containment (memory dump, log files)
- Document timeline: When alert occurred, who detected it, what it shows
- Note: Do NOT shut down systems yet unless active data exfiltration is ongoing

---

### 3.3 Containment Authority & Decision Points

**Who decides if containment is needed:**

Incident Commander decides whether to immediately isolate/block systems based on:
- Severity classification
- Nature of threat (active vs. dormant)
- Business impact of containment vs. continued compromise

**Containment decisions:**
- **CRITICAL with active exfil:** Isolate affected systems immediately (network disconnect, power off if necessary)
- **CRITICAL with ransomware:** Isolate immediately, do not pay ransom without legal/insurance approval
- **HIGH:** Containment may wait for investigation to confirm breach, unless risk is unacceptable
- **MEDIUM/LOW:** Investigate before containment (may be false positive)

**Documentation:** Every containment decision and its rationale must be logged (for investigation and board reporting).

---

## Part 4: Roles During Detection & Initial Response

### Clear Decision Authority

**Who makes what decisions:**

| Decision | Who Decides | Escalation if disagreement |
|----------|------------|---------------------------|
| Is this an incident? | Detection Lead | CISO |
| What severity level? | Detection Lead | IC / CISO |
| Immediate containment needed? | Incident Commander | CISO |
| Notify customers? | Communications Lead + Legal | CISO / CEO |
| Notify law enforcement? | CISO / Legal | CEO / Board |
| Public disclosure? | CISO / CEO / Legal | Board / Insurance |
| Investigation scope/timeline? | IC / Forensics Lead | CISO |
| System restoration? | IC / Systems Lead | CISO / CEO (cost/business impact) |

---

### Communication Cadence

**During CRITICAL incident:**
- Initial escalation call: Within 15 minutes
- Full IRT call: Within 1 hour
- Leadership briefing: Every 4 hours until status stabilizes
- Board notification: As needed, within 24 hours for confirmed breach

**During HIGH incident:**
- Initial escalation: Within 1 hour
- IRT coordination: As needed (may be email/chat if not complex)
- Leadership update: Daily or as major decisions arise
- Board notification: Only if data compromise confirmed

**During MEDIUM incident:**
- No formal coordination required initially
- Update leadership only if severity increases

---

## Part 5: Integration with Board & Executive Leadership

### 5.1 Board Escalation Criteria

**Board must be notified (via CISO or CEO) immediately if:**
1. CRITICAL severity incident with confirmed CUI data compromise
2. Incident likely to require regulatory notification (breach notification law)
3. Incident affecting ability to fulfill customer contracts
4. Incident with potential media exposure or public relations impact
5. Incident requiring legal response or law enforcement involvement

**Board notification includes:**
- What happened (incident type and timing)
- What data is at risk (CUI? PII? How much?)
- Current status (contained? Still active?)
- Expected business/financial impact
- Next steps and timeline for resolution
- Regulatory/legal implications

**Follow-up board communication:**
- First board meeting after incident: Full briefing
- Subsequent meetings: Progress update on investigation and remediation

---

### 5.2 Executive Authority During Incident

**CEO approves:**
- Public disclosure of incident
- Notification to law enforcement
- Ransom payment decisions (with insurance)
- Major business decisions (take systems offline? Stop services? etc.)
- Legal strategy for breach notification

**CISO approves:**
- Technical containment/response actions
- Scope of investigation
- Timeline and resources for remediation
- External vendor engagement (forensics, incident response)
- Notification to customers (in coordination with Communications)

**CFO approves:**
- Budget allocation for incident response and remediation
- Insurance claim submission
- Costs associated with investigation/recovery
- Customer notification/remediation costs

**Legal counsel approves:**
- Breach notification strategy and timing
- Communications to regulators or law enforcement
- Document preservation and litigation hold
- Vendor agreements for incident response

---

## Part 6: Supporting Systems & Documentation

### 6.1 Incident Tracking System

All incidents (even LOW severity) must be logged with:
- Incident ID (date + sequence number, e.g., INC-2026-06-001)
- Title and description
- Severity classification and justification
- Detection method and detection time
- Incident Commander assigned
- Team members engaged
- Status (Open / Investigating / Contained / Closed)
- Timeline of key events
- Root cause (once determined)
- Lessons learned (once completed)

**Retention:** Minimum 7 years (federal contractor requirement for incident records)

---

### 6.2 Incident Communication Log

Document all communications about the incident:
- Internal notifications (who was told, when, via what method)
- Escalations to leadership
- Board notifications
- Customer notifications
- Law enforcement/regulatory notifications
- Each with: timestamp, recipient, content summary, approval authority

**Retention:** Minimum 7 years

---

### 6.3 Evidence Preservation Procedures

**Chain of custody:** Any evidence collected must be documented:
- What was collected (file, log, memory dump, etc.)
- When it was collected
- Who collected it and who handled it
- How it was stored (secure location, access controlled)
- Who had access and when

**Forensic protocol:** If external forensics firm will be engaged:
- Do not investigate suspected breach yourself (may compromise evidence)
- Photograph system state if possible (before touching)
- Preserve system logs and memory if feasible
- Isolate suspected systems to prevent further compromise

**Legal holds:** If incident may result in litigation:
- Preserve all evidence (emails, logs, communications) related to incident
- Communicate litigation hold to all employees
- Prevent destruction or modification of evidence


