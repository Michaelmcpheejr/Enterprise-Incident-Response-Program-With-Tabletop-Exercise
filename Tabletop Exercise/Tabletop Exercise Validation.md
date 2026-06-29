# Enterprise Incident Response Program
## Tabletop Exercise Validation: Gap Discovery & Remediation

**Purpose:** Validate IR procedures through realistic scenario-based exercises; identify gaps and implement improvements

**Format:** Three comprehensive tabletop scenarios (Ransomware, Data Exfiltration, Supply Chain Attack) showing actual gap discovery, impact assessment, and remediation planning

**Participants:** IR team, CISO, Operations, IT Security, Finance, Legal, Board representative

**Duration:** Three separate half-day exercises (one per scenario)

---

## TABLETOP SCENARIO 1: RANSOMWARE ATTACK

**Scenario Setup:** Friday 2:30 PM, user reports files changed to .locked extension across file share

**Timeline of Exercise:**

---

### SCENARIO 1, PHASE 1: INITIAL DETECTION & CONTAINMENT (0-60 minutes)

**2:30 PM - Alert Received:**
- Detection Lead receives report: "All files on shared drive changing to .locked"
- Incident Commander immediately activated
- Phase 1 severity classification: CRITICAL (active encryption detected)

**2:35 PM - Isolation Decision:**
- IC decides to immediately isolate file servers (active spreading)
- Network team disconnects file server 1 and 2 from production network
- IC orders: "Do NOT reboot systems, preserve memory"

**2:45 PM - Initial Investigation:**
- Team identifies ransomware process still running on file server 1
- Command: tasklist /v shows "enc.exe" consuming 95% CPU (ransomware encrypting files)
- Task killed: taskkill /IM enc.exe /F (stops encryption, doesn't decrypt)

**3:00 PM - Backup Check:**
- Ops team checks backup system status
- Backup server is ALSO isolated (encrypted .locked files detected)
- IC says: "Our backup is compromised. We can't restore from this."
- Problem identified: No verified clean backup available

---

### GAP #1 DISCOVERED: No Isolated Recovery Environment Pre-Staged

**[X] Finding (How discovered):**
During containment phase, team attempted recovery planning. Realized:
- All systems share same production network
- Backup system was also on production network (encrypted by ransomware)
- Recovery systems (if rebuilt) would be vulnerable to re-infection
- No air-gapped recovery environment exists for safe restoration

**Exercise dialogue:**
- IC: "What's our backup integrity? Can we restore safely?"
- Ops: "Backup is encrypted. All our backups are on the same network."
- IT: "If we rebuild file servers on the production network, ransomware could spread again during recovery."
- IC: "We have no recovery environment. We're vulnerable during restoration."

**[Y] Affects (Business/operational impact):**
- Recovery time: 8-12 hours (manual rebuild + verification) vs. 2-3 hours with isolated environment
- Re-infection risk: HIGH (ransomware persistence mechanisms could remain, re-infect during rebuild)
- Data loss: If no backup is clean, manual recovery from air-gapped backup needed (older data)
- CUI compliance: Can't guarantee "clean" recovery for CUI-containing systems without isolated environment

**[Z] Best Remediation:**

**Best practice solution:**
Build isolated recovery environment (segmented network, cannot communicate with production):
- Infrastructure cost: $40,000 (estimates vary based on system complexity; range typically $30K-$60K)
- Labor: 120 hours for design/build/testing (estimates vary based on system complexity)
- Timeline: 90 days (estimates vary; larger environments may require 120+ days)
- Success criteria: Full system recovery in <4 hours, verified clean

**Interim workaround (implemented immediately):**
Pre-stage offline backups on dedicated hardware drives, kept in secured air-gapped storage:
- Cost: $0 (use existing hardware)
- Setup: 2 hours
- Testing: Verify backup restoration on isolated system weekly
- Limitation: Backups must be regularly verified, add 1-2 hours to recovery process

**Procedure update (Phase 1):**
- Add requirement: Test backup integrity monthly (full restore on isolated system)
- Add decision point: "If backup is compromised, activate offline backup recovery procedure"
- Add timeline assumption: "Recovery 8-12 hours without isolated environment; 2-3 hours with isolated environment"

**Minor Realism Note:**
This scenario assumes organization lacked offline backup strategy and all backups resided on production network (common among mid-market federal contractors with limited infrastructure budgets). Organizations with mature backup practices (offline backups, air-gapped copies) would avoid this gap. This scenario illustrates worst-case but realistic situation.

---

### SCENARIO 1, PHASE 2: RECOVERY ATTEMPTS (60-120 minutes into exercise)

**3:30 PM - Recovery Planning:**
- IC: "We need to restore systems. What are our options?"
- Ops: "We have yesterday's backup, but it's encrypted. We also have a 1-week-old offline backup."
- Finance: "Restoring to 1-week-old data means losing Friday's work. What's the business impact?"
- Business owner: "We can accept Friday's data loss, but need systems operational by Monday."
- IC: "Can we rebuild manually by Monday?"
- IT: "We can have file servers operational by Sunday, but without testing all applications."

**3:45 PM - Backup Verification Confusion:**
- Ops team says they'll restore from "known-good backup"
- IC asks: "How do we know it's clean? Do we have a procedure to verify?"
- Ops: "We... don't have a formal verification procedure. We'll just try to restore and see if it works."
- CISO: "That's risky. We could restore and re-infect immediately."
- IC: "We need a backup verification procedure before we restore from anything."

---

### GAP #2 DISCOVERED: Backup Integrity Verification Procedure Missing

**[X] Finding (How discovered):**
During recovery planning, team realized they had no defined procedure to verify backup integrity:
- No process for testing backup restoration on isolated system
- No checksum verification of backup files
- No procedure for detecting ransomware in backup before restoring to production
- Assumption: "Backups are probably clean" (risky assumption)

**Exercise dialogue:**
- IC: "How do we verify the 1-week-old backup is clean before restoring?"
- Ops: "Uh... we don't have a procedure for that."
- CISO: "We need to test restoration first. What if the backup is corrupted or has malware?"
- IT: "We'd need to restore to an isolated system, scan for malware, then move to production."
- IC: "That adds 4-6 hours to recovery. We don't have a process for this."

**[Y] Affects (Business/operational impact):**
- Recovery delay: 4-6 hours added (restore to isolated system, scan, verify clean, restore to production)
- Re-infection risk: HIGH (restoring unverified backup could re-introduce ransomware)
- Investigation capability: No evidence of whether backup contains malware or is truly clean
- Regulatory compliance: Can't demonstrate CUI data integrity without backup verification

**[Z] Best Remediation:**

**Best practice solution:**
Implement quarterly backup verification testing:
- Procedure: Monthly restore of backup to isolated test environment
- Verification: Scan restored systems for malware, verify file integrity
- Timeline: 2-4 hours per monthly test (estimates vary based on backup size and system complexity)
- Cost: ~10 hours labor per month (estimates vary based on automation level)
- Success criteria: Documented evidence that backup is clean and restorable

**Procedure immediately implemented:**
- Backup verification checklist created (3 steps)
- Step 1: Restore backup to isolated test environment (not production network)
- Step 2: Run malware scans and integrity checks on restored data
- Step 3: If clean, document verification; if infected, go to offline backup
- Testing schedule: Monthly starting this month

**Added to Phase 1 decision point:**
- "If backup verification shows corruption/malware: Do not restore; go to offline backup or accept data loss"

---

### SCENARIO 1, PHASE 3: RANSOM DECISION (120-150 minutes into exercise)

**4:30 PM - Ransom Note Discovered:**
- Forensics team finds ransom note on file server: "All your files are encrypted. Pay $500,000 in Bitcoin to recover."
- Note includes: "Reply here [email], Payment deadline: Monday 5 PM"
- CFO received email: "We got a ransom demand for $500K"

**4:35 PM - Ransom Decision Confusion:**
- CFO: "Should we pay? Insurance might cover this."
- CISO: "We haven't even determined if restoration from backup is viable yet. Let's not rush into paying."
- CEO (via phone): "Who decides whether we pay ransom? Is this CISO's call or Finance?"
- IC: "We don't have a defined procedure for this. We should escalate to the Board."
- Board Chair (via phone): "You're escalating ransom decisions to the Board during active incident? That seems slow."
- Legal: "FBI recommends we don't pay ransom, but business pressure might make us want to."
- 15-minute debate: Who decides? CISO? CFO? Insurance broker? Board?

---

### GAP #3 DISCOVERED: Ransom Decision Authority Unclear

**[X] Finding (How discovered):**
When ransom demand arrived, no defined procedure existed for who decides whether to pay:
- CISO could argue "It's a security decision, let me decide"
- CFO could argue "It's a financial decision, I own this"
- CEO could argue "It's a business decision, I'm responsible"
- Board could argue "This is material risk, needs Board approval"
- Insurance broker says "Your policy requires you to contact us first"
- 15-minute delay in decision-making during time-sensitive incident

**Exercise dialogue:**
- CFO: "Insurance covers cyber events. Should I authorize payment to insurance?"
- CISO: "Before we pay anything, we need to know if backup restoration is viable."
- CEO: "How much does insurance cover? Can they negotiate with the attacker?"
- Insurance broker: "You must notify us within 24 hours of knowing you're breached. Payment decisions need our approval."
- Board Chair: "You should escalate material financial decisions to the Board."
- Legal: "FBI recommendation is don't pay ransom; it funds criminal activity."
- IC (frustrated): "I need a clear decision authority so we can move forward."

**[Y] Affects (Business/operational impact):**
- Decision delay: 15+ minutes debating who decides (in real incident, time = money)
- Compliance risk: Missing required notifications (Insurance, FBI, Board)
- Financial risk: Paying ransom without proper approval chain
- Reputation risk: If we pay ransom, becomes public knowledge; looks like we're funding criminals

**[Z] Best Remediation:**

**Best practice solution:**
Define ransom decision authority in advance (before incident happens):
- Document: Who has authority to decide ransom payment?
- Sequence: CISO → CFO → CEO → Board (if amount exceeds $X)
- Insurance requirement: Notify insurance within 24 hours
- FBI requirement: Notify FBI before paying ransom
- Timeline: Must decide within 24 hours of ransom demand

**Procedure immediately implemented:**
- Ransom Decision Authority Matrix created (CISO always provides technical assessment; final decision authority escalates by amount):
  - **<$50K:** CISO + CFO decide together
    - CISO provides: Recovery feasibility, timeline, risk assessment
    - CFO provides: Insurance implications, financial impact
    - Decision makers: CISO + CFO (joint approval required)
  
  - **$50K-$500K:** CISO + CFO + CEO decide together, notify Board
    - CISO provides: Technical assessment (can we recover? How long?)
    - CFO provides: Insurance coverage, financial impact, cost-benefit
    - CEO provides: Business impact, reputation risk, decision authority
    - Decision makers: CEO (final call) with CISO + CFO input (both must concur it's feasible)
  
  - **>$500K:** CISO + CFO + CEO brief Board; Board + CEO decide together
    - CISO provides: Technical assessment to Board (recovery timeline, risks)
    - CFO provides: Financial impact, insurance coverage implications
    - CEO/Board: Final decision on whether to pay
    - Board authority: For amounts exceeding $500K, Board must formally approve decision

- **Key principle:** CISO must participate in AND concur with every ransom decision
  - CISO assessment is mandatory input (not optional)
  - If CISO recommends recovery is feasible without paying: Default to recovery path
  - If CISO recommends recovery is impossible or will take >7 days: Then consider ransom
  - CISO must be convinced paying ransom is justifiable

- Notification checklist: Insurance (24 hr), FBI (before payment), CEO (immediate), Board (within 4 hours)
- Pre-negotiated contacts: Insurance broker, FBI field office, FBI Cyber Division

**Added to Phase 3 playbook (1.6 Escalation Triggers):**
- New trigger: "Ransom demand arrives → CISO immediately assesses recovery feasibility → Follow Ransom Decision Authority Matrix with CISO as mandatory participant"

**Minor Realism Note:**
Ransom decisions are inherently complex and high-stress. The Authority Matrix provides structure, but real incidents involve significant debate between Security, Finance, Legal, and Executive teams. FBI recommends not paying ransom (it funds criminal enterprise), but business realities may differ. The CISO's technical assessment (recovery feasibility, timeline, risk) should be the primary input to the decision. This scenario reflects realistic organizational tensions during ransom demands.

---

### SCENARIO 1, RECOVERY & INVESTIGATION (150+ minutes into exercise)

**5:00 PM - Forensics Deep Dive:**
- Forensics team finds ransomware persistence mechanism: SSH key installed on backup server
- Finding: "Attacker created backdoor SSH key. If we restore, they can re-infect immediately."
- Impact: Can't simply restore and recover; must verify attacker is completely evicted first

**5:15 PM - Process Termination Problem:**
- IC asks: "Can we terminate the ransomware process?"
- IT: "We already killed enc.exe, but are there other persistence mechanisms?"
- Forensics: "Yes, there's a scheduled task that re-launches encryption every 30 minutes."
- IT: "We don't have a documented procedure for identifying and terminating all ransomware processes."
- CISO: "What's our standard procedure for finding ransomware processes? Where's the checklist?"
- IT team searches: "We don't have one. We just kill obvious processes and hope we got them all."

---

### GAP #4 DISCOVERED: No Process Termination Procedure Documented

**[X] Finding (How discovered):**
During investigation, team realized they had no systematic procedure for identifying ALL ransomware processes:
- Killed obvious process (enc.exe) but missed persistence mechanisms
- Missed scheduled task that re-launches encryption
- Missed SSH backdoor key (not a process, but persistence)
- No checklist for "What processes does ransomware typically create?"
- No procedure for verifying "We got all of them"

**Exercise dialogue:**
- IC: "Can we guarantee all ransomware is dead?"
- IT: "We killed enc.exe... but we're not sure if there are others."
- Forensics: "There's a scheduled task that re-launches encryption. We need to disable that too."
- IT: "Where's our procedure for finding all ransomware processes?"
- IT (searching through documentation): "We don't have one written down."
- IC: "This is dangerous. We could restore from backup and get immediately re-infected."

**[Y] Affects (Business/operational impact):**
- Re-infection risk: HIGH (missed persistence mechanisms mean attacker can re-infect after restore)
- Recovery failure: System appears restored but encryption resumes hours later
- Investigation delay: Each time we miss a process, recovery takes longer
- Trust degradation: After restore, system gets re-infected → team loses confidence in response

**[Z] Best Remediation:**

**Best practice solution:**
Create comprehensive ransomware process termination procedure:
- Checklist: Standard ransomware indicators (process names, registry keys, scheduled tasks, startup scripts)
- Persistence hunting: Procedure to identify backdoors, SSH keys, new accounts
- Verification: Before declaring ransomware dead, verify against checklist
- Timeline: 30 minutes per system to complete thorough termination verification (estimates vary based on system complexity and malware sophistication)
- Cost: Document existing knowledge, format as checklist (2-4 hours)

**Procedure immediately implemented:**
- Ransomware Persistence Hunting Checklist created:
  - Step 1: Identify main encryption process (tasklist, process explorer)
  - Step 2: Search for scheduled tasks launching encryption (schtasks /query /v)
  - Step 3: Search for SSH keys/backdoor accounts (registry keys, user accounts)
  - Step 4: Search startup locations (registry, startup folders)
  - Step 5: Document all findings; verify each termination
- Forensics Lead to review checklist, add technique-specific items

**Added to Phase 3 playbook (1.3 Containment Procedures, Step 3):**
- Replace vague "Stop the encryption" with detailed checklist
- "For each affected system, follow Ransomware Persistence Hunting Checklist to ensure complete termination"

---

## TABLETOP SCENARIO 2: DATA EXFILTRATION/BREACH

**Scenario Setup:** Tuesday 2:45 PM, DLP alert: "User account 'john.smith@company.com' transferred 2.3 GB to external IP"

**Timeline of Exercise:**

---

### SCENARIO 2, PHASE 1: DETECTION & IMMEDIATE RESPONSE (0-60 minutes)

**2:45 PM - DLP Alert:**
- Security alert: "2.3 GB data transferred from john.smith@company.com to IP 203.0.113.47"
- Files flagged: Customer list (PII), contract files (CUI), technical specifications

**3:00 PM - Initial Investigation:**
- john.smith account accessed from IP 203.0.113.47 (Eastern Europe, suspicious)
- john.smith's normal login is from Miami office (he's home today, working from coffee shop)
- File access logs show: john.smith accessed customer directory (236 files) in 12 minutes
- Network logs show: 2.3 GB uploaded to external IP in 18 minutes (very fast, bulk transfer)
- Conclusion: Account compromised, data exfiltrated

**3:15 PM - Containment:**
- Reset john.smith password immediately
- Revoke VPN session (forced logoff)
- Block destination IP 203.0.113.47 at firewall
- Check for other suspicious activity on john.smith account (none detected in past 24 hours, so first incident)

**3:30 PM - Scope Assessment Chaos:**
- IC asks: "How much PII was affected?"
- Legal: "We need to know if we have to notify anyone."
- Finance: "What's the business impact?"
- IR team searches: "Okay, we know he accessed customer list... but how many records?"
- Operations: "We have 150,000 customer records total."
- IR: "But did he download all 150,000 or just some of them?"
- Ops: "We... don't have a procedure to determine that. We'd have to manually count files he accessed."
- Finance: "Can we assume worst case (all 150,000)?"
- CISO: "No. We need to know what actually exfiltrated, not guess."
- 30 minutes spent manually reviewing file access logs to determine actual scope

---

### GAP #1 DISCOVERED: No Defined Procedure for Identifying Affected Data Scope

**[X] Finding (How discovered):**
When determining scope of data exfiltration, no systematic procedure existed:
- Manual review of file access logs to determine "which files actually accessed?"
- No automated tool to correlate file access with exfiltration timeframe
- No procedure to classify data in customer list (is it PII? How many people affected?)
- No procedure to identify which customers need notification
- Manual counting took 30+ minutes; prone to error

**Exercise dialogue:**
- Legal: "How many individuals were affected? We need this for breach notification law."
- IR: "Let me manually count files in the customer list..."
- Ops: "Each customer record has 15-20 fields of PII per person. If we downloaded 500 records, that's 7,500-10,000 individuals affected."
- Finance: "What's our notification liability? $100-$500 per person affected?"
- CISO: "We need a procedure to automatically determine data scope, not manual searching."

**[Y] Affects (Business/operational impact):**
- Notification delay: 30+ minutes spent on manual investigation
- Inaccurate scope: Risk of missing affected individuals (underestimate) or overcounting (panic)
- Breach notification law compliance: Must notify within 30 days of knowing about breach; delays start the clock ticking
- Regulatory liability: If scope is inaccurate, regulators may penalize
- Financial impact: Estimated $100-$500 per affected individual

**[Z] Best Remediation:**

**Best practice solution:**
Build automated data classification and scope determination system:
- Data tagging: All data repositories tagged with data type (PII, CUI, public)
- Access logging: Track which data each user accesses
- Exfiltration mapping: Automated tool correlates access logs with network exfiltration to determine "what left"
- Timeline: 2-3 weeks to implement (estimates vary based on data complexity and system integration)
- Cost: 80-120 hours of development and testing (estimates vary based on automation level)

**Interim workaround (implemented immediately):**
Create manual investigation procedure and checklist:
- Step 1: Identify exfiltration timeframe (start time and end time of data transfer)
- Step 2: Pull file access logs for that timeframe from affected account
- Step 3: For each file accessed, classify: PII/CUI/Public
- Step 4: Count individuals affected (e.g., if customer list has 1 record per person, count records)
- Step 5: Document scope: "X files accessed, Y individuals affected, Z records in exfiltrated data"
- Owner: Detection Lead to complete within 1 hour of incident detection
- Accuracy: ~95% (manual process, but systematic)

**Added to Phase 2 procedures (2.4 Impact Assessment):**
- New checklist for "Data Scope Identification Procedure"
- Step-by-step guide for correlating file access with exfiltration
- Timing assumption: "Scope determination takes 30 minutes to 1 hour"

---

### SCENARIO 2, PHASE 2: BREACH NOTIFICATION (60-120 minutes into exercise)

**4:15 PM - Notification Decision:**
- Legal: "We have exfiltrated 237 customer records (PII). We must notify affected individuals."
- Finance: "Who decides the timeline? 30 days from 'knowing about breach' or from 'discovery'?"
- CISO: "We discovered this today. Timeline starts now."
- CEO (via phone): "Can we negotiate with attacker? Maybe recover the data?"
- Legal: "No. We follow notification law, not attacker demands. But who sends the notification?"
- Communications: "We draft the notification, but who approves the content?"
- PR: "Media will find out about this. Should we do public statement?"
- 20-minute debate: Who decides on notification timeline? Who approves notification language?

---

### GAP #2 DISCOVERED: Breach Notification Timeline and Decision Authority Unclear

**[X] Finding (How discovered):**
When it came time to notify customers of potential breach, no procedure defined:
- Who decides notification timeline? (Legal says 30 days, business wants to delay)
- Who approves notification content? (Legal, CISO, CEO, PR all have opinions)
- What channel? Email, certified mail, phone call? (Law varies by state)
- What exactly do we say? (Avoid legal liability while informing customers)
- When does the clock start? (At discovery or at confirmation of breach?)

**Exercise dialogue:**
- Legal: "Breach notification law requires notification within 30 days of knowing about breach."
- CFO: "Can we delay notification while investigating further?"
- Legal: "Delay could expose us to regulatory penalties."
- CISO: "Who decides the notification approach? Who approves the message?"
- Communications: "We need sign-off from Legal and PR before sending anything."
- CEO: "I want to review any customer notification personally."
- Lawyer: "That creates liability. CEO shouldn't review notification (conflict of interest)."
- IC (frustrated): "We need clear decision authority for this."

**[Y] Affects (Business/operational impact):**
- Notification delay: 30+ minutes debating who decides (should be pre-determined)
- Regulatory penalty risk: Missing 30-day window = fines from regulators
- Legal liability: Unclear approval authority = risky communications
- Customer trust: Delayed notification = customers find out via other means first (worse)
- PR risk: Media finds out before we notify customers = reputation damage

**[Z] Best Remediation:**

**Best practice solution:**
Define breach notification authority and timeline in advance:
- Policy: "Breach notification within 30 days of discovery or as required by law (whichever is sooner)"
- Authority: Legal + CISO approve notification content; CEO/Board approves notification strategy
- Timeline: Notification drafted within 24 hours; approved within 48 hours
- Process: Escalate to Board if notification will be public (media-facing)
- Communication channels: Email (default), certified mail (for high-risk), phone (for executive contacts)
- Cost: Policy document only (no implementation cost)

**Procedure immediately implemented:**
- Breach Notification Authority Matrix created:
  - <100 individuals affected: Legal + CISO approve notification
  - 100-1,000 individuals: Add CFO approval (financial impact assessment)
  - >1,000 individuals: Escalate to Board, include PR review
- Notification timeline: "Approved notification drafted within 24 hours; sent within 48 hours"
- Notification content checklist: What to say, what NOT to say (legal review required)
- Regulatory coordination: Who notifies FBI/regulators (Legal responsible)

**Added to Phase 3 playbook (2.6 Escalation Triggers):**
- New trigger: "Breach confirmed → Immediately notify Legal and follow Breach Notification Authority Matrix"

**Minor Realism Note:**
Breach notification decisions involve multiple stakeholders with competing interests: Legal wants to minimize liability, Communications wants to control narrative, Finance worries about costs, CISO wants accurate scope. This scenario reflects real-world tension. The Authority Matrix provides structure but requires strong CISO/Legal partnership. Federal contractors must notify if CUI is exposed (regulatory requirement); notification timeline is typically 30 days from discovery but varies by state law for PII.

---

## TABLETOP SCENARIO 3: SUPPLY CHAIN ATTACK

**Scenario Setup:** Wednesday 10:00 AM, vendor security team notifies: "We were breached; attackers accessed your systems"

**Timeline of Exercise:**

---

### SCENARIO 3, PHASE 1: VENDOR NOTIFICATION & IMMEDIATE RESPONSE (0-60 minutes)

**10:00 AM - Vendor Breach Notification:**
- Third-party vendor calls: "Our infrastructure was compromised last week. Attackers may have accessed customer systems."
- Vendor: "We had backdoor access for 72 hours. We're still investigating, but wanted to warn you."
- CISO: "Which of our systems could they have accessed through your account?"
- Vendor: "We have credentials to your file server and database. We also have SSH access to three servers."
- CISO: "Were your credentials used during the breach?"
- Vendor: "We're not sure yet. That's why we're calling everyone."

**10:15 AM - Scope Assessment:**
- IC: "We need to know: Did attackers use vendor credentials to access our systems?"
- Operations: "The vendor has access to... let me check. They have VPN account, SSH keys on servers 5/7/9, and database account."
- IT: "To revoke vendor access, I need to: Disable VPN user, remove SSH keys from 3 servers, disable database account, remove from file share..."
- IT: "How long will that take?"
- IT (searching documentation): "We don't have a procedure for this. I'll have to manually check each system."
- CISO: "Start immediately. In the meantime, I need to pull vendor access logs."

**10:30 AM - Access Revocation Begins:**
- IT: "Removing SSH keys from server 5... done. Server 7... done. Server 9..."
- IT: "Wait, server 9 has SSH backdoors installed. Were those from vendor or from attacker?"
- Forensics: "Pull the SSH logs. When were those keys installed?"
- IT: "The logs show new keys installed 3 days ago. That matches vendor breach timeline."
- CISO: "So attacker used vendor SSH access to create backdoors for persistence?"
- Forensics: "Looks like it. We need to rebuild server 9."

**11:00 AM - Vendor Access Still Being Revoked:**
- IT (after 1 hour of manual work): "Done revoking vendor access from servers, file share, VPN, and database."
- IT: "But I might have missed something. I had to manually check each system. No centralized inventory."
- CISO: "How long did this take?"
- IT: "1 hour for 12 systems. If we had more systems, it could take 3-4 hours."
- CISO: "Next time we need a faster process."

---

### GAP #1 DISCOVERED: No Vendor Access Revocation Procedure Existed

**[X] Finding (How discovered):**
When vendor account needed immediate revocation, no centralized procedure existed:
- Vendor had access scattered across multiple systems (VPN, SSH, database, file share)
- IT had to manually navigate to each system to revoke access
- No centralized access management system (no single place to disable vendor account)
- Process took 1 hour to complete; prone to missing systems

**Exercise dialogue:**
- CISO: "How do we revoke vendor access from ALL systems at once?"
- IT: "We... don't have a centralized way. I have to do it manually on each system."
- IT: "VPN revocation is different than SSH key removal, which is different than database disable."
- IT: "There's no single place to revoke everything at once."
- CISO: "That's dangerous. It took 1 hour and we might have missed something."

**[Y] Affects (Business/operational impact):**
- Time to revoke: 1 hour (without centralized IAM) vs. 15 minutes (with centralized IAM)
- Miss risk: HIGH (might miss some systems, attacker keeps partial access)
- Attacker dwell time: 1 hour window where compromised vendor account still had access
- Lateral movement: Attacker could use vendor account to access additional systems during revocation delay
- CUI exposure: If vendor account had CUI access, 1 hour of potential unauthorized access

**Remediation timeline variance (as discussed):**
- **Without centralized IAM:** Up to 3-4 hours to locate and revoke vendor access across distributed systems (vendor credentials stored in individual system configs, documented in spreadsheets, access scattered across multiple platforms)
- **With centralized IAM system (Active Directory, Okta):** Up to 120 minutes (revoke at identity provider, propagates to all connected systems; single source of truth for vendor access inventory)
- **During tabletop:** Organization had no centralized IAM; revocation took 1 hour of manual coordination across 12 systems

**[Z] Best Remediation:**

**Best practice solution:**
Implement centralized Identity & Access Management (IAM) system:
- Single identity provider (Okta, Azure AD, or similar)
- All vendor accounts provisioned through central system
- Vendor revocation propagates to all connected systems automatically
- Timeline: 4-6 months to implement and migrate (estimates vary based on environment complexity)
- Cost: $60,000-$100,000 (licensing + implementation; estimates vary based on system scale)
- Benefit: Vendor revocation drops from 1 hour to 15 minutes

**Interim workaround (implemented immediately):**
Create vendor access inventory and revocation checklist:
- Document: Which vendors have access to which systems?
- Inventory: Central spreadsheet listing all vendor accounts and access types
- Procedure: One-page revocation checklist (steps to revoke from each system type)
- Timeline: Annual review of inventory; quarterly access verification
- Owner: IT Security responsible for maintaining inventory
- Revocation time: 30-45 minutes with checklist (vs. 1 hour without documentation), but prevents missing systems

**Added to Phase 3 playbook (3.3 Containment Procedures, Step 1):**
- "Immediately disable vendor account on all systems using Vendor Access Revocation Checklist"
- Checklist specifies: VPN revocation, SSH key removal, database account disable, file share removal, application access removal
- Timeline: "15 minutes with centralized IAM; 30-45 minutes with manual revocation using checklist"

**Minor Realism Note:**
This scenario assumes organization without centralized IAM, which is common among mid-market federal contractors. Vendors often have credentials scattered across multiple systems (VPN, SSH, database, file share, applications). Revocation without centralized system requires manual navigation to each platform. Organizations with mature IAM platforms (Active Directory, Okta) can revoke in 15-20 minutes. The 1-hour timeline reflects worst-case distribution of vendor credentials across 12+ systems without documentation.

---

### SCENARIO 3, PHASE 2: VENDOR ASSESSMENT (60-120 minutes into exercise)

**11:30 AM - Vendor Security Assessment:**
- CISO calls vendor: "What's your security posture? How did you get breached?"
- Vendor: "Employees had weak passwords. Attacker brute-forced accounts."
- CISO: "How can we trust your systems? What security improvements are you making?"
- Vendor: "We're... uh... implementing password policy updates."
- CISO: "That's not sufficient. We need you to demonstrate security capability before we re-enable your access."
- Vendor: "We don't have formal security certifications. We're a small company."
- CISO: "But you have access to our CUI systems. We need vendor security assessment procedures."
- CISO: "This is a gap in our program. We should have vetting procedure before vendors access critical systems."

---

### GAP #2 DISCOVERED: Vendor Security Assessment Requirements Not Defined

**[X] Finding (How discovered):**
When reviewing vendor security posture after breach, no defined assessment requirements existed:
- No procedure for assessing vendor security capability before granting access
- No annual re-certification requirement
- No standards for vendor security (certifications, security controls, etc.)
- No checklist for "Is this vendor trustworthy?"

**Exercise dialogue:**
- CISO: "Before we give them CUI access again, we need to verify their security."
- Compliance: "Do we have vendor security requirements?"
- CISO: "Not formally. We just grant access and hope they're secure."
- CISO: "This is risky. We should require vendors to prove security capability."
- Compliance: "What would we even check? ISO 27001? SOC 2?"
- CISO: "We need to define vendor vetting criteria. This is a gap."

**[Y] Affects (Business/operational impact):**
- Risk unknown: We don't know if vendor is security-conscious or cavalier
- Breach risk: Vendor security weakness becomes our vulnerability
- Compliance risk: If vendor breach leads to our CUI exposure, auditors blame us for not vetting vendors
- Regulatory exposure: NIST 800-171 requires third-party risk management; we had none

**[Z] Best Remediation:**

**Best practice solution:**
Implement Vendor Security Assessment Program:
- Vendor requirements: ISO 27001 certification (or equivalent) OR SOC 2 Type II audit
- Annual re-certification: Require vendor to provide updated certification annually
- Pre-access vetting: Before granting access to CUI systems, verify vendor certification
- Incident notification: Require vendors to notify within 24 hours of breach
- Access review: Quarterly review of which vendors have access to CUI
- Timeline: 30 days to establish vendor vetting procedure
- Cost: Administrative (no licensing cost)

**Procedure immediately implemented:**
- Vendor Security Assessment Checklist created:
  - Has vendor passed ISO 27001 audit? (or equivalent)
  - Does vendor have incident response plan?
  - Will vendor notify us within 24 hours of breach?
  - Does vendor have data protection procedures (encryption, access controls)?
  - Annual re-certification required (before contract renewal)
- Pre-access approval: CISO must approve vendor access to CUI systems (must pass assessment)
- Ongoing monitoring: Quarterly vendor security review (any changes to their security posture?)

**Added to Phase 3 playbook (3.5 Recovery Procedures, Step 5):**
- "Before re-enabling vendor access, require vendor to demonstrate security improvements"
- "Require vendor security assessment; document findings in CISO approval file"

**Minor Realism Note:**
Most organizations begin their TPRM (Third-Party Risk Management) programs after experiencing a vendor-related incident. This scenario reflects common federal contractor practice: vendor security assessment requirements often emerge post-incident rather than being defined proactively. NIST 800-171 Section 3.14 (Supply Chain Risk Management) explicitly requires vendor assessment, but many mid-market contractors lack formal procedures. This gap represents typical organizational maturity level before incident triggers remediation.

---

## SUMMARY: TABLETOP EXERCISE FINDINGS

**Total gaps identified across 3 scenarios: 8 gaps**

**Ransomware Scenario Gaps (4):**
1. ✅ No isolated recovery environment pre-staged
2. ✅ Backup integrity verification procedure missing
3. ✅ Ransom decision authority unclear
4. ✅ No process termination procedure documented

**Data Exfiltration Scenario Gaps (2):**
1. ✅ No defined procedure for identifying affected data scope
2. ✅ Breach notification timeline and decision authority unclear

**Supply Chain Scenario Gaps (2):**
1. ✅ No vendor access revocation procedure existed
2. ✅ Vendor security assessment requirements not defined

---

## REMEDIATION STATUS & TRACKING

**All gaps remediated or in progress:**

| Scenario | Gap # | Gap Title | Priority | Status | Due Date | Owner |
|----------|-------|-----------|----------|--------|----------|-------|
| Ransomware | 1 | Recovery environment | CRITICAL | In progress | 09-15-2026 | Infrastructure |
| Ransomware | 2 | Backup verification | CRITICAL | Complete | 07-15-2026 | Operations |
| Ransomware | 3 | Ransom authority | HIGH | Complete | 07-01-2026 | CISO |
| Ransomware | 4 | Process termination | HIGH | Complete | 07-10-2026 | Forensics |
| Data Exfil | 1 | Data scope procedure | HIGH | Complete | 07-15-2026 | Detection |
| Data Exfil | 2 | Breach notification | CRITICAL | Complete | 07-01-2026 | Legal/CISO |
| Supply Chain | 1 | Access revocation | HIGH | In progress (IAM eval) | 12-31-2026 | IT |
| Supply Chain | 2 | Vendor assessment | HIGH | Complete | 07-20-2026 | Compliance |

---

## TABLETOP EXERCISE VALUE

**What the organization learned:**
- ✅ Procedures work, but have friction points (e.g., backup verification delay)
- ✅ Gaps are real and have real consequences (e.g., 1-hour revocation vs. 15-minute ideal)
- ✅ Collaboration is critical (CISO/Finance/Legal must work together smoothly)
- ✅ Decision authority must be pre-defined (ransom, notification, approval)
- ✅ Tools matter (centralized IAM would eliminate 45 minutes of manual work)

**Program improvements:**
- 4 Phase 1 procedures updated with new decision points
- 2 Phase 2 procedures updated with new investigation techniques
- 3 Phase 3 playbooks updated with tactical procedures and checklists
- 1 new Phase 4 procedure: Annual vendor security re-certification

**Next steps:**
- Implement all remediation actions on timeline
- Conduct second round of tabletop exercises in 90 days
- Measure improvement (revocation should drop from 1 hour to 30 min with checklist)
- Plan for full IAM implementation in 2027

