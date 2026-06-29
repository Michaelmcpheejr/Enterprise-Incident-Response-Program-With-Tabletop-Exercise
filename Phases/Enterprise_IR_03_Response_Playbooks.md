# Enterprise Incident Response Program - Phase 3
## Response Playbooks

**Role:** Incident Commander, Containment & Response Lead, Investigation Lead  
**Primary Standard:** NIST SP 800-61 Rev. 2 (Containment & Recovery phases)  
**Goal:** Execute threat-specific response procedures to contain, investigate, and recover  
**Timeline:** CRITICAL incidents require playbook execution within first 30-60 minutes  
**Output:** Incident contained, investigation initiated, recovery underway

---

## Overview: Using Playbooks

A playbook is a **pre-planned, tested response procedure** for a specific threat type.

**When to use playbooks:**
- Incident is CRITICAL or HIGH severity
- Threat type matches one of the playbooks below (ransomware, data exfil, supply chain)
- Phase 2 investigation has confirmed the attack type
- Incident Commander authorizes playbook execution

**Playbook structure:**
1. **Detection Indicators** - How you confirm this threat type
2. **Immediate Actions** (0-15 minutes) - Stop the threat
3. **Containment Procedures** (15-60 minutes) - Prevent spread
4. **Investigation Focus** - What to look for while containing
5. **Recovery Procedures** - How to restore systems
6. **Escalation Triggers** - When to escalate within the playbook
7. **Critical Decisions** - What IC must decide

**Important:** Playbooks are guides, not scripts. Incident Commander adapts procedures based on actual incident circumstances. Flexibility matters more than rigid procedure following.

---

## PLAYBOOK 1: RANSOMWARE RESPONSE

### 1.1 Detection Indicators: Confirm Ransomware

**You have ransomware if:**

✅ **Ransom note appears** on affected systems (clearest indicator)

✅ **File extensions change** suddenly across multiple files (e.g., all .docx become .locked)

✅ **File sizes increase** dramatically (encryption adds overhead)

✅ **Backup files deleted or encrypted** (attacker prevents recovery)

✅ **Files become unreadable** by normal applications (corruption or encryption)

✅ **System slowdown** during encryption (heavy I/O activity)

✅ **Network activity spike** to command-and-control server (ransomware communicating)

**Investigation approach:**
- Check affected systems for ransom note (location: Desktop, Documents, file share root)
- Verify file extension changes across multiple file types
- Check backup/shadow copy status (get-wmiobject win32_shadowcopy on Windows)
- Review process execution logs for encryption tools (look for suspicious .exe or script execution)

**If uncertain,** escalate to Incident Commander before proceeding. False alarm is better than treating non-ransomware as ransomware.

---

### 1.2 Immediate Actions (0-15 Minutes)

**Incident Commander makes go/no-go decision for containment.**

**Step 1: Activate ransomware playbook (within 5 minutes of confirmation)**
- Incident Commander authorizes execution
- Notify all team members: IR is now in ransomware response mode
- Update incident tracking system: "Ransomware playbook ACTIVE"

**Step 2: Identify affected systems (within 10 minutes)**
- Which systems show signs of encryption?
- How many systems are affected? (1 vs. 10 vs. 100+ changes scope)
- Are affected systems critical to business operations?

**Step 3: Decision point: Immediate isolation needed?**
- YES if: Ransomware is actively spreading (new systems getting encrypted in real-time)
- YES if: Backup systems show encryption (attacker attacking recovery target)
- NO if: Encryption appears dormant (no active spreading detected)

**If YES (active spreading):**
- Immediately isolate affected systems from network (physical disconnect or network block)
- Do NOT shut down systems (need to preserve memory for investigation)
- Document which systems isolated and at what time

**If NO (dormant):**
- Do not isolate yet (preserve evidence, allow investigation to proceed)
- Monitor network traffic for signs of re-activation
- Prepare isolation option if spreading resumes

---

### 1.3 Containment Procedures (15-60 Minutes)

**Goal: Stop encryption from spreading to unaffected systems.**

**Step 1: Network segmentation (15-20 minutes)**
- Isolate affected systems from file servers and backups
- Block lateral movement: Remove network access for affected systems
- Preserve evidence: Do not shut down systems, just disconnect from network

**Step 2: Backup preservation (20-30 minutes)**
- Immediately take backup system offline (disconnect from network)
- Verify backup integrity: Can you restore from backup? (test on isolated system)
- If backup is compromised, you may need to recover from older backup (adds time)

**Step 3: Stop the encryption (30-40 minutes)**
- For each affected system, identify the ransomware process
- Identify processes spawning encryption activity
- Terminate processes (taskkill /IM process.exe /F on Windows)
- Note: Terminating process does NOT decrypt files, only stops further encryption

**Step 4: Preserve system state (40-50 minutes)**
- Do NOT reboot systems (rebooting destroys ransomware process memory)
- Capture running processes (tasklist /v, pslist)
- Capture network connections at time of isolation
- Photograph ransom note and encrypted files
- Collect memory dump if forensics firm is involved

---

### 1.4 Investigation Focus During Containment

**While containment is underway, investigation team answers:**

**Attack vector:** How did ransomware get in?
- Phishing email with attachment?
- Exploit of unpatched system?
- Compromised remote access credentials?
- Supply chain (software update containing malware)?

**Entry point:** Which system was initially compromised?
- Trace back from encryption spread
- May not be the system with first visible encryption

**Persistence:** Did ransomware establish persistence?
- Will it re-activate when systems are restored?
- Was a backdoor installed for continued access?

**Scope:** How many systems are actually affected?
- Only user workstations, or also servers?
- Were critical systems (domain controller, backup) reached?

**Timeline:** When did encryption start?
- Earliest evidence of ransomware
- How long was it active before detection?

---

### 1.5 Recovery Procedures

**Recovery is decided by Incident Commander based on:**
- Backup availability and integrity
- Restoration timeline vs. business impact
- Ransom demands (do NOT pay without C-suite/insurance approval)

**Option A: Restore from backup (preferred)**

**Prerequisite:** Backup is verified uncompromised and sufficiently recent

**Steps:**
1. Rebuild affected systems from known-good backup
2. Start with critical systems first (domain controller, file servers)
3. Restore user workstations once servers are restored
4. Verify systems are operational before restoring to production
5. Test: Confirm systems are responsive, applications work, data is intact

**Timeline:** 4-8 hours depending on system count and backup size

**Risk:** If backup contains dormant ransomware, you re-infect during restore

---

**Option B: Manual rebuild (if backup unavailable or compromised)**

**Steps:**
1. Document system configuration before rebuilding (what applications, what settings?)
2. Rebuild OS from clean installation media
3. Restore applications from source (CD, deployment server, not from backup)
4. Restore data from backup (only if backup verified uncompromised)
5. If data backup is compromised, consider recovery from air-gapped backup or accepting data loss

**Timeline:** 16-40 hours depending on number of systems

**Risk:** Time-consuming, but more assurance that you're not re-infecting

---

**Option C: Accept data loss and rebuild without old data**

**Used when:**
- Backup is completely compromised
- Backup timeline is too old (recovering to week-old data is unacceptable)
- Business decides time-to-recovery is more critical than data recovery

**Steps:**
1. Rebuild OS and applications cleanly
2. Restore only current data from recent, verified source (not main backup)
3. Accept that data modified since last verified backup is lost

**Timeline:** 4-8 hours

**Risk:** Data loss, customer impact, potentially regulatory implications if CUI data is lost

---

### 1.6 Escalation Triggers Within Playbook

**Escalate to Incident Commander immediately if:**

1. **Encryption spreads to critical systems** - Domain controller, backup servers, file servers
   - Action: IC decides whether to take entire network offline (nuclear option)
   - Business impact: All systems down, no access to data

2. **Backup is compromised** - Investigation reveals backup is encrypted
   - Action: IC must decide recovery approach (manual rebuild or accept data loss)
   - Business impact: Determines timeline and cost

3. **Ransom demand arrives** - Attacker sends ransom note with ransom amount
   - Action: IC briefs CEO/CFO/Insurance, legal advises on options
   - Business impact: Financial decision ($50K to $10M+ depending on organization)

4. **Ransomware is actively re-spreading** - New systems getting encrypted after initial containment
   - Action: IC escalates to CISO, may implement full network isolation
   - Business impact: Containment failed, need more aggressive action

5. **Investigation reveals supply chain compromise** - Ransomware came from trusted vendor/partner
   - Action: Escalate to executive sponsor (CISO/CEO), notify partner
   - Business impact: Broader incident than initially thought

---

### 1.7 Ransomware Playbook: Gaps Discovered in Tabletop

**During ransomware tabletop exercise, three gaps were identified:**

**Gap 1: No isolated recovery environment pre-staged**
- **Discovery:** Playbook Step 1.5 (Recovery) assumes rebuild capability, but during tabletop we realized all systems share same network, so recovery environment would be re-infected
- **Impact:** Recovery time adds 8-12 hours; re-infection risk is high
- **Remediation:** [Described in tabletop document]

**Gap 2: Backup integrity verification procedure missing**
- **Discovery:** Playbook Step 1.5 says "verify backup integrity" but actual verification procedure was undefined
- **Impact:** Team didn't know HOW to verify backups, so risked restoring from compromised backup
- **Remediation:** [Described in tabletop document]

**Gap 3: Ransom decision authority unclear**
- **Discovery:** Playbook Step 1.6 says "IC briefs CEO/CFO/Insurance" but didn't specify who decides if ransom is paid
- **Impact:** During incident, 30-minute delay while executives debated authority
- **Remediation:** [Described in tabletop document]

**Gap 4: No process termination procedure documented**
- **Discovery:** Playbook Step 1.3 says "terminate processes" but didn't include actual commands or tools
- **Impact:** Team wasn't sure which processes to terminate or how
- **Remediation:** [Described in tabletop document]

---

## PLAYBOOK 2: DATA EXFILTRATION/BREACH RESPONSE

### 2.1 Detection Indicators: Confirm Data Exfiltration

**You have data exfiltration if:**

✅ **Bulk data transfer detected** - NetFlow shows large volume to external IP (100 MB+ in short timeframe)

✅ **Unusual outbound connections** - Proxy logs show connections to non-standard ports (443 to cloud storage, 22 to external server)

✅ **File access anomaly** - Large number of files accessed by single user in short timeframe (e.g., 1,000 files in 30 minutes)

✅ **Data staging activity** - Files zipped/compressed before transfer (indicates intentional exfiltration)

✅ **DLP alert triggered** - Data loss prevention system detects sensitive data (CUI) being transmitted

✅ **Endpoint logging shows data export** - USB drive activity, cloud upload, email attachment of sensitive files

**Investigation approach:**
- Pull network logs (NetFlow, firewall, proxy) for timeframe of suspected exfiltration
- Identify destination IP addresses and calculate data volume
- Check file access logs for which files were accessed
- Determine user account used for access
- Look for data staging (temporary files created before export)

**If uncertain,** investigate thoroughly before concluding exfiltration. False alarm is possible (legitimate backup, data migration, etc.).

---

### 2.2 Immediate Actions (0-15 Minutes)

**Incident Commander decides: Is this exfiltration or false positive?**

**Step 1: Confirm exfiltration (within 5 minutes)**
- Pull network logs showing outbound transfer
- Identify destination IP (internal cloud service vs. external IP)
- Calculate data volume (is it significant or routine?)
- Check DLP logs if available (what data was flagged?)

**Step 2: Identify affected data (within 10 minutes)**
- What files were accessed/transferred?
- Is CUI involved? (Regulatory notification trigger)
- Is PII involved? (Breach notification law trigger)
- How many individuals affected?

**Step 3: Identify compromised account (within 10 minutes)**
- Which user account performed the access/transfer?
- Was account compromised (unauthorized use) or authorized (legitimate user)?
- If compromised: Attacker had access for how long?
- If authorized: Is this a policy violation or authorized access?

**Step 4: Decision point: Lock down account immediately?**
- YES if: Account is confirmed compromised, attacker is still accessing systems
- YES if: Need to prevent further exfiltration
- NO if: Need to continue investigation and monitor

---

### 2.3 Containment Procedures (15-60 Minutes)

**Goal: Stop data from leaving organization; preserve evidence of what left.**

**Step 1: Block exfiltration channels (15-20 minutes)**
- Block destination IP at firewall (prevent further outbound connections)
- Revoke remote access credentials if account was compromised
- Reset password for compromised account (force re-authentication if attacker reconnects)
- Disable email forwarding rules (if exfiltration was email-based)
- Block cloud upload services at proxy (if data was exfiltrated to cloud)

**Step 2: Preserve evidence of exfiltration (20-30 minutes)**
- Capture complete network logs for exfiltration timeframe (NetFlow, firewall, proxy)
- Capture file access logs for affected systems
- Photograph DLP alerts and log entries
- Document data volume transferred (bytes, file count)
- Identify destination IP geolocation (where did data go?)

**Step 3: Secure compromised account (30-40 minutes)**
- Reset password
- Revoke all active sessions (force logoff)
- Check for persistence (did attacker create backdoor account?)
- Review recent account activity (what systems were accessed?)

**Step 4: Assess ongoing risk (40-50 minutes)**
- Is attacker still active (new connections attempts)?
- Are other accounts showing suspicious activity?
- Has attacker established persistence (backdoor for future access)?

---

### 2.4 Investigation Focus During Containment

**While containment is underway, investigation team answers:**

**Data identification:** What exactly left the organization?
- File names, file sizes, data types (technical specs, customer list, etc.)
- CUI? PII? Trade secrets? Public data?
- How many files? How much total data volume?

**Breach scope:** How many individuals affected?
- If PII exfiltrated: How many people?
- Required for breach notification law determination

**Timeline:** When did exfiltration occur?
- First data transfer timestamp
- Last data transfer timestamp
- Total duration

**Destination:** Where did data go?
- IP address geolocation
- Service used (cloud storage, email, FTP, etc.)
- Can data be recovered (request from recipient)?

**Account access:** How long was account compromised?
- First suspicious activity
- Last suspicious activity
- What other systems did attacker access?

---

### 2.5 Recovery Procedures

**Data exfiltration recovery focuses on investigation completion and breach notification.**

**Step 1: Determine if recovery is possible**
- Can you identify where exfiltrated data went?
- Can you request deletion from recipient (unlikely for malicious actors)?
- For legitimate third-party recipients: Send formal request to delete data

**Step 2: Complete breach scope determination**
- Phase 2 investigation determines: Exactly what data left?
- Affects breach notification decision (do you have to notify?)

**Step 3: Prepare breach notification (if required)**
- Notify affected individuals (if PII exfiltrated)
- Notify law enforcement and U.S. Cert (if CUI exfiltrated)
- Notify customers (if customer data exfiltrated)
- Timeline: As quickly as possible, typically within 30 days

**Step 4: Root cause remediation (Phase 4)**
- How did account get compromised?
- Implement control to prevent recurrence (MFA, better access controls, etc.)

---

### 2.6 Escalation Triggers Within Playbook

**Escalate to Incident Commander immediately if:**

1. **CUI data confirmed exfiltrated** - Regulatory notification is required
   - Action: IC briefs legal and CISO; breach notification plan initiated
   - Business impact: Public disclosure likely, regulator notification mandatory

2. **Large volume of PII exfiltrated** - Breach notification law triggered
   - Action: IC initiates breach notification procedures
   - Business impact: Customer notification, potential legal liability

3. **Attacker is still active** - Ongoing exfiltration attempts detected
   - Action: IC escalates to CISO; may implement more aggressive containment
   - Business impact: Threat still present, more data at risk

4. **Data destination is identifiable and recoverable** - Unusual case where attacker uploaded to accessible location
   - Action: IC may request law enforcement to seize data from attacker
   - Business impact: Potential recovery of stolen data

5. **Investigation reveals insider threat** - Employee deliberately exfiltrated data
   - Action: IC escalates to HR and legal; potential criminal investigation
   - Business impact: Employee termination, potential prosecution

---

### 2.7 Data Exfiltration Playbook: Gaps Discovered in Tabletop

**During data exfiltration tabletop exercise, two gaps were identified:**

**Gap 1: No defined procedure for identifying affected data scope**
- **Discovery:** Playbook Step 2.2 says "identify affected data" but procedure to determine WHAT data exfiltrated was missing
- **Impact:** During tabletop, team spent 45 minutes debating which files were accessed; no systematic way to identify scope
- **Remediation:** [Described in tabletop document]

**Gap 2: Breach notification timeline and decision authority unclear**
- **Discovery:** Playbook Step 2.5 mentions breach notification but didn't specify who decides, what triggers notification, and timeline
- **Impact:** During tabletop, regulatory notification question caused 30-minute discussion with no clear decision point
- **Remediation:** [Described in tabletop document]

---

## PLAYBOOK 3: SUPPLY CHAIN ATTACK RESPONSE

### 3.1 Detection Indicators: Confirm Supply Chain Compromise

**You have supply chain compromise if:**

✅ **Malware traced to third-party software** - Malware analysis reveals origin is from vendor/partner software

✅ **Vendor account unauthorized activity** - Vendor credentials used to access systems outside normal business hours or scope

✅ **Software update contains malware** - Update from trusted vendor contains malicious code

✅ **Third-party vendor breach notification** - Partner notifies you they were breached and your systems may be affected

✅ **Unusual lateral movement from vendor-controlled system** - Attack originates from system granted to vendor, spreads to your systems

**Investigation approach:**
- Pull audit logs for vendor account activity (when did they access? What did they access?)
- Analyze malware (if present) to determine origin
- Review vendor access logs against known vendor work windows
- Contact vendor to confirm if they were breached
- Review software integrity (verify vendor software version, patch level)

---

### 3.2 Immediate Actions (0-15 Minutes)

**Incident Commander decides: Is this vendor-related or unrelated incident?**

**Step 1: Identify which vendor/partner is involved (within 5 minutes)**
- Malware attributed to which vendor?
- Vendor account showing suspicious activity?
- Which vendor's software is affected?
- Contact vendor: "We detected unauthorized activity from your account/software; were you compromised?"

**Step 2: Assess breach scope (within 10 minutes)**
- How much access did vendor have to your systems?
- Which systems can vendor access?
- What data repositories is vendor connected to?
- Is CUI accessible through vendor connection?

**Step 3: Determine attack vector (within 10 minutes)**
- Did attacker use compromised vendor account?
- Did attacker use malware in vendor software?
- Is vendor's infrastructure compromised (attacker controls vendor's supply chain)?

**Step 4: Decision point: Immediate isolation of vendor access needed?**
- YES if: CUI systems are at risk or attacker is still actively using vendor access
- YES if: Vendor account shows ongoing unauthorized activity
- NO if: Attack is dormant and can be investigated before isolation

---

### 3.3 Containment Procedures (15-60 Minutes)

**Goal: Sever compromised vendor connection; preserve evidence.**

**Step 1: Revoke vendor access (15-20 minutes)**
- Immediately disable vendor account on all systems
- Revoke VPN access, API keys, SSH keys
- Remove vendor from file share permissions
- Terminate active sessions

**Step 2: Audit vendor's access history (20-35 minutes)**
- What systems did vendor access during compromise window?
- What data repositories did vendor access?
- How long was vendor account active?
- What did vendor do while accessing your systems? (file access, data transfer, etc.)

**Step 3: Check for persistence (35-45 minutes)**
- Did attacker (using vendor credentials) install backdoors?
- Were new accounts created for persistent access?
- Are there scheduled tasks or startup scripts installed by vendor/attacker?
- If persistence found: Systems cannot be trusted until remediated

**Step 4: Gather intelligence for forensics analysis (45-60 minutes)**
- Engage forensics team; provide them with initial analysis framework:
  - What version of vendor software is installed? Compare to known-good version
  - Are software signing certificates valid? (Document any certificate anomalies)
  - Timeline: When was software installed/updated? When did suspicious activity begin?
  - Scope: Which systems have this vendor software installed?
- This upfront intelligence allows forensics team to focus deep analysis on specific malware characteristics rather than starting from scratch
- Forensics team may identify other team members who can assist with specific analysis tasks based on this framework
- Document all findings for vendor notification (other customers may need warning)

---

### 3.4 Investigation Focus During Containment

**While containment is underway, investigation team answers:**

**Vendor breach scope:** Was vendor compromised or did vendor introduce malware intentionally?
- Does vendor acknowledge breach?
- Are other vendor customers affected?
- When was vendor compromised?

**Access history:** What did attacker do using vendor access?
- Which systems were accessed?
- Which data was accessed?
- Was data exfiltrated?
- Were lateral movement attempts made?

**Malware analysis:** What is the malware and who created it?
- Malware functionality (what does it do?)
- Attribution (nation-state, cybercriminal, hacktivist?)
- Is malware likely to affect other customers?

**Timeline:** When did compromise occur?
- When was vendor first compromised?
- When did malware first appear in your systems?
- How long was attacker active?

---

### 3.5 Containment & Recovery Procedures

**Supply chain attacks typically require vendor notification and remediation verification.**

**Step 1: Notify vendor immediately**
- "We detected activity from your account/software; are you aware of this?"
- Share indicators of compromise
- Request their remediation plan

**Step 2: Audit systems accessed by vendor**
- List all systems where vendor had access
- Check each for signs of compromise
- Prioritize: Check CUI-accessible systems first

**Step 3: Verify vendor's remediation**
- Vendor must patch/fix their software
- Require vendor to provide patch and evidence of fix
- Test patch in isolated lab environment before deploying

**Step 4: Reimploy vendor access (once verified safe)**
- After vendor provides remediated software/credentials
- After your team verifies remediation
- Re-enable vendor access with additional monitoring
- Monitor vendor activity closely for 30 days post-remediation

**Step 5: Update vendor management procedures (Phase 4)**
- Add vendor security assessment requirements
- Require quarterly vendor security re-certification
- Implement continuous monitoring of vendor account activity

---

### 3.6 Escalation Triggers Within Playbook

**Escalate to Incident Commander immediately if:**

1. **CUI systems accessed via vendor account** - Regulatory notification may be required
   - Action: IC briefs legal; assess if breach notification needed
   - Business impact: Potential regulator notification

2. **Multiple customers affected** - Vendor's security incident affects more than just you
   - Action: Coordinate with vendor on public disclosure
   - Business impact: Public relations implications

3. **Vendor is unresponsive** - Vendor doesn't acknowledge breach or won't provide remediation plan
   - Action: IC decides whether to continue vendor relationship
   - Business impact: May need to terminate vendor and find alternative

4. **Persistence mechanisms discovered** - Attacker installed backdoors using vendor access
   - Action: Full remediation required; rebuild affected systems
   - Business impact: Systems must be rebuilt after vendor remediation verified

5. **Attacker still using vendor access** - Ongoing unauthorized activity from vendor account
   - Action: Immediate full isolation; don't wait for vendor response
   - Business impact: Active threat, more aggressive containment needed

---

### 3.7 Supply Chain Attack Playbook: Gaps Discovered in Tabletop

**During supply chain attack tabletop exercise, two gaps were identified:**

**Gap 1: No vendor access revocation procedure existed**
- **Discovery:** Playbook Step 3.3 says "immediately disable vendor account" but actual procedure to revoke access across all systems was missing
- **Impact:** Remediation timeline varies significantly based on IAM maturity:
  - **Without centralized IAM:** Up to 3-4 hours to locate and revoke vendor access across distributed systems (vendor credentials stored in individual system configs, documented in spreadsheets, access scattered across multiple platforms)
  - **With centralized IAM system (Active Directory, Okta):** Up to 120 minutes (revoke at identity provider, propagates to all connected systems; single source of truth for vendor access inventory)
  - **During tabletop:** Organization had no centralized IAM; revocation took 3.5 hours of manual coordination across 12 systems
- **Remediation:** [Described in tabletop document]

**Gap 2: Vendor security assessment requirements not defined**
- **Discovery:** Playbook Step 3.5 mentions "vendor security assessment" but organization had no documented vendor vetting process
- **Impact:** Vendor security posture was unknown; couldn't assess if vendor was compromise risk
- **Remediation:** [Described in tabletop document]


