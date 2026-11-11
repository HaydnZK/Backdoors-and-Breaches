# Resonant Destruction
## Scenario
We're looking at a Medium-High alert on a financial workstation (WKSTN-FIN-042) for user jsmith. An unverified binary in a user profile tried accessing a core OS process, right alongside a suspicious burst of external login attempts against a corporate service portal.

### Security Alert Details
- Alert Name: Anomalous Process Interaction and External Authentication Burst
- Severity: Medium-High
- Source Host: WKSTN-FIN-042
- User Context: CORP\jsmith (Standard User)
- Observed Activity: Non-standard handle creation requested against a core operating system process by an unverified binary running from a user profile directory. Correlated edge logs show a brief cluster of rapid login attempts from an external IP address shortly before the endpoint event.

### Alert Description
An endpoint monitoring agent flagged an unusual process interaction involving high-privilege access rights on a financial workstation. Simultaneously, perimeter gateway logs recorded a short burst of repeated authentication requests originating externally against a corporate service portal for the same user account. Further analysis of endpoint telemetry and authentication logs is required to determine the scope of activity.

---

## Scenario Cards
Initial Compromise: Credential Stuffing
Pivot/Escalate: LSASS Credential Dump
C2/Exfil: Unauthorized Filesharing Utility
Persistence: Additional Credential Creation 

---

## Employee Interview Rolls *Needs Update*

| D20 Roll | Outcome Category | Result & Narrative |
| --- | --- | --- |
| 1 | Lose Detection | The employee gets extremely defensive and demands HR representation. The ensuing red tape completely blocks your investigation. The Incident Master removes one available detection method from the team roster. |
| 2-5 | Hint 1 | The employee recalls a strange interaction. "I got an unexpected alert on my phone asking me to approve a multi-factor login prompt while I was away from my desk eating lunch." |
| 6 | Draw Starting Condition | Rumors of a cyber incident spread quickly through the regional shipping terminals causing operational panic. Draw a Starting Condition card to represent the chaos. |
| 7-9 | Hint 2 | A staff member complains about a weird computer glitch. "My workstation had a command prompt window pop up and run background commands right after I booted up this morning." |
| 10 | Draw Consultant | An executive overhears the interview and remembers they have an active incident response retainer with a cybersecurity firm. Draw a Consultant card to assist the investigation. |
| 11-12 | Hint 3 | An employee mentions an unusual system prompt. "A background process kept asking for administrative credentials while I was trying to check container manifests yesterday." |
| 13 | Draw Inject | The interview accidentally tips off a compromised user who might be monitored by the attacker. The adversary accelerates their timeline. Draw an Inject card. |
| 14-15 | Hint 4 | A logistics coordinator notes some annoying background activity. "My cloud file sync utility has been constantly spinning and uploading massive archives in the background even though I didn't save any new shipping documents." |
| 16-19 | Remove Card | The interview provides solid evidence that a specific attack vector was absolutely not used. The Incident Master removes one stale detection card from the board that would not have revealed an active scenario card. |
| 20 | Discover Detection | Critical breakthrough! The employee remembers a crucial detail or hands over a specific file path they noticed earlier. The Incident Master reveals one detection method that will directly uncover an active scenario card. |

---

## Relevant Details
- Who We Are: The Internal IT and Security Operations team for Meridian Global Freight, a mid-sized international supply chain logistics and customs brokerage firm specializing in maritime container tracking, cross-border freight forwarding, and automated customs filing.

- What We Do: Manage logistics scheduling platforms, bills of lading, customs clearance records, manifests, and billing networks that keep physical cargo moving across international shipping lanes.

- Workforce & Management Model: A mix of corporate headquarters staff, regional warehouse and port terminal IT administrators, and a large contingent of logistics coordinators who frequently collaborate with external shipping agents, port authorities, and transport carriers.

- Identity Management: Hybrid identity setup combining local Active Directory domain controllers for legacy warehouse management applications with cloud-based enterprise platforms utilizing single sign-on.

- Infrastructure Layer: A mix of on-premises data closets at regional shipping hubs running virtualization clusters for logistics databases, paired with cloud repositories handling billing workflows and global client portals.

- Collaboration & Messaging Protocols: Frequent reliance on corporate email, vendor portals, and shipping-specific data exchange standards for communication with external freight forwarders, carriers, and customs agencies.

- The Environment Baseline: High-frequency data transfers involving shipping documents, manifests, and billing invoices, combined with constant remote access authentication requests from regional logistics managers and port terminal operators working across various operational time zones.

---

## Card Reveal Details
### Initial Compromise: Credential Stuffing
```txt
Apr 14 08:12:44 vpn-gw01 auth[4102]: Failed password for invalid user jsmith from 198.51.100.42 port 52144 ssh2
Apr 14 08:12:44 vpn-gw01 auth[4102]: Failed password for invalid user jsmith from 198.51.100.42 port 52146 ssh2
Apr 14 08:12:45 vpn-gw01 auth[4102]: Accepted publickey for jsmith from 198.51.100.42 port 52148 ssh2: RsaKey ID 42
```
What we're looking at: This is an edge gateway authentication log showing a credential stuffing attack pattern. Notice the rapid sequence of failed login attempts from a single source IP address (`198.51.100.42`) immediately followed by a successful authentication event. Rather than exploiting a software vulnerability, the attacker leveraged automated tooling to spray known username and password combinations against an external-facing portal lacking multi-factor enforcement, successfully establishing a user-level session.

- The incoming connection traces back to a known commercial proxy network rather than a standard corporate travel location or local residence.
- Subsequent log analysis reveals that immediately after the successful login, internal query activity began scanning local domain structures, setting the stage for privilege extraction.

### Pivot/Escalate: LSASS Credential Dump
```txt
EventCode: 10
UtcTime: 2026-04-14 08:14:12.892
SourceProcessGuid: {a1b2c3d4-e5f6-7890-abcd-ef0123456789}
SourceImage: C:\Windows\System32\rundll32.exe
TargetImage: C:\Windows\System32\lsass.exe
GrantedAccess: 0x1F0FFF
```
What we're looking at: This is an endpoint log showing a living-off-the-land credential harvesting technique. The process `rundll32.exe` executed the built-in Windows library `comsvcs.dll` and its `MiniDump` function to target `lsass.exe`. Notice the `GrantedAccess` code requesting full memory read rights. By abusing a native system binary rather than dropping custom malware, the attacker extracted hashed credentials and tokens directly from system memory to elevate their operational privileges.

- Tracing the process lineage shows it originated from the active user session established during the initial perimeter gateway compromise.
- Endpoint file system telemetry indicates temporary output directories were created immediately following memory access, anticipating bulk data handling.

### C2/Exfil: Unauthorized Filesharing Utility
```txt
2026-04-14 08:35:21 INFO [1] rclone: Go-root/1.22 - os/arch: windows/amd64 - os/version: 10.0.19045
2026-04-14 08:35:22 DEBUG [1] rclone: Using config file "C:\Users\jsmith\AppData\Local\rclone\rclone.conf"
2026-04-14 08:35:25 INFO [1] archive_staging_04.7z: Transferring to remote storage target mega://financial_vault_backup
```
What we're looking at: This is a cloud access proxy log showing unapproved data exfiltration. The attacker is leveraging a legitimate commercial file syncing utility (`rclone`) running from a user profile directory to push compressed archives to an external cloud storage provider (`api.mega.co.nz`). Notice the massive asymmetric transfer with over 148 megabytes sent compared to minimal bytes received. Because the tool communicates over standard encrypted HTTPS, basic perimeter firewalls view this as normal traffic rather than malicious data loss.

- Process execution logs confirm the synchronization tool was launched using administrative tokens harvested earlier via low-level memory interactions on core system processes.
- Network flow analysis notes concurrent secondary connections linking back to auxiliary authentication attributes attached to the user account outside normal provisioning windows.

### Persistence: Additional Credential Creation 
```txt
EventCode: 4738
Account Name: jsmith
Account Domain: CORP
Changed Attributes:
  - User Account Control: 
    - Don't Require Preauthentication: Enabled
  - Service Principal Names: Restored/Modified
```
What we're looking at: This is a Security Event log showing modifications to an existing user account. Rather than provisioning a brand-new user from scratch, the attacker silently attached auxiliary security credentials and alternate authentication parameters to a legitimate enterprise account. Because the changes leverage native administrative permissions, the updates bypassed standard IT provisioning workflows, ensuring the adversary retains reliable access even if the initial entry vector is patched.

- Audit logs show the administrative session modifying these account attributes originated directly from the external IP address tied to the perimeter gateway compromise.
- Following the addition of these alternate authentication parameters, host telemetry records an increase in automated background data bundling and outbound transfer tasks.

---

## Heat Raisers
### Normal
Turn 2: Staging and Escalation
Help desk receives reports of unusual login bursts against external gateways using valid credentials, shortly followed by internal alerts flagging low-level process memory reading on key domain machines.

Turn 4: Executive Pressure
Admins notice unapproved file synchronization utility processes executing from standard user profiles, quietly bundling and compressing local directories, while C-suite leadership demands an immediate operational briefing.

Turn 6: Third-Party Inquiries
Identity audits reveal that auxiliary security credentials and alternate authentication methods have been silently attached to existing, legitimate user accounts without passing through standard IT provisioning. Legal and compliance reps reach out asking whether an undisclosed breach protocol has been initiated.

Turn 8: Infrastructure Instability
Outbound network monitoring detects heavy, sustained data transfers routed through commercial cloud storage sync tools. Core authentication services experience intermittent latency as background tasks process compressed archives.

Turn 10: The Climax and Leadership Lifeline
1. Ransomware payloads deploy across critical infrastructure, file shares freeze, ransom notes populate desktop backgrounds, and a leak site countdown timer goes live.
2. Executive leadership steps in with emergency authorization and dedicated resources. The defenders pick one of the following options to make their final stand:
  - Desperate Blitz: Pick 3 Procedure cards, roll all 3 simultaneously, and take the raw d20 rolls at face value.
  - Focused Execution: Pick 1 Procedure card and apply a plus 5 bonus to the roll.
  - Executive Intervention: The Incident Commander picks 3 Procedure cards, consisting of 1 correct card matching an active attack phase and 2 incorrect red herrings. The defenders choose 1 of those 3 cards and roll with zero modifiers, ignoring all scenario debuffs.


### Live
Turn 0: Initial Foothold: Private Event: The threat actor executes Credential Stuffing against an exposed edge portal and quickly follows up with LSASS Credential Dump on an internal host.
- Effect: Available for Defenders to discover starting now: Initial Compromise & Pivot/Escalate.

Turn 1: SIEM Clearing & Log Tampering: Sabotage: Akira runs commands to clear security event logs and disrupt ingestion pipelines, creating gaps in centralized log correlation.
- Effect: -2 to next SIEM Log Analysis roll

Turn 2: Expansion & Data Staging: Private Event: The threat actor deploys an Unauthorized Filesharing Utility to stage sensitive data for exfiltration and establishes Additional Credential Creation in Active Directory to secure redundant access.
- Effect: Available for Defenders to discover starting now: C2/Exfil & Persistence.

Turn 3: Living off the Land: Sabotage: The attackers hijack legitimate administrative user contexts and run native Windows administrative tools, muddying behavioral baselines and blending into normal daily operational traffic.
- Effect: -2 to next UEBA roll

Turn 4: Executive Distraction: Public Event: Targeted executive phishing and hostile communications hit leadership directly. Executive leadership calls an emergency briefing, pulling key personnel out of the loop and preventing the team from using Crisis Management for this turn.
- Effect: Crisis Management unusable this turn

Turn 5: Telemetry Blinding: Sabotage: Akira unloads or tampers with endpoint telemetry drivers using compromised admin privileges, blinding local agent monitoring on critical systems.
- Effect: -2 to next Endpoint Analysis roll

Turn 6: Legal & Retainer Freeze: Public Event: Corporate Legal places a temporary freeze on external communications while reviewing non-disclosure scope and retainer protocols, blocking the team from using Call a Consultant for this turn.
- Effect: Call a Consultant unusable this turn

Turn 7: Encrypted Noise: Sabotage: Attackers wrap outbound traffic in heavy encryption and tunnel communication over common web protocols, diluting PCAP analysis and hiding within normal network volume.
- Effect: -2 to next Network Threat Hunting roll

Turn 8: Administrative Deadlock: Public Event: Group Policy modifications and conflicting access control updates create an administrative deadlock on core infrastructure, preventing the defenders from executing network-wide segment drops or using Isolation procedure cards for this turn.
- Effect: Isolation unusable this turn

Turn 9: Edge Device Blinding: Sabotage: Akira floods edge gateways with high-frequency garbage requests and alters firewall logging configs to truncate records, burying true connection logs under heavy noise.
- Effect: -2 to next Firewall Log Review roll

Turn 10: Climax & Leadership Lifeline
- Public Attacker Event: Ransomware payloads deploy simultaneously across critical infrastructure, file systems freeze, .txt ransom notes drop across network shares, and a leak site countdown timer appears on screen.
- Leadership Event (The Lifeline): Executive leadership grants emergency authorization and dedicated resources. The defenders pick one of the following options to make their final stand:
  - Desperate Blitz: Pick 3 Procedure cards, roll all 3 simultaneously, and take the raw d20 rolls at face value (no modifiers).
  - Focused Execution: Pick 1 Procedure card and apply a +5 bonus to the roll.
  - Executive Intervention: The Incident Commander picks 3 Procedure cards (1 correct card matching an active attack phase, 2 incorrect red herrings). The defenders choose 1 of those 3 cards and roll with zero modifiers, ignoring all scenario debuffs.

---

## Final Story
Weeks before the incident, the operators set their sights on Meridian Global Freight, targeting their hybrid infrastructure to slip past defenses. They launched an automated credential stuffing campaign against an exposed edge VPN portal, quickly hitting a valid username and password combination that lacked proper multi-factor enforcement. Once inside the perimeter, they wasted no time moving to the local workstation. Instead of dropping noisy malware, they used native Windows tools, invoking rundll32 to call the memory dump function in comsvcs.dll against the lsass process. This allowed them to extract cached credentials and elevate their operational privileges straight from volatile memory while evading basic signature checks.

To blind the security team from their initial movements, the operators ran commands to clear security event logs and disrupt the centralized log forwarding pipelines, creating immediate blind spots in the SIEM. With the environment quiet, they deployed an unauthorized file synchronization utility to bundle and stage sensitive customs brokerage records and billing files. At the same time, they silently attached auxiliary security credentials and alternate authentication parameters directly to existing enterprise accounts in Active Directory, locking in redundant access that bypassed standard IT provisioning workflows.

Moving deeper into the network, the operators lived off the land by hijacking legitimate administrative user contexts and running native administrative tools to blend seamlessly into normal daily operational traffic. As executive distractions and administrative deadlocks slowed down the internal response, the attackers blinded local endpoint agents and tunneled their traffic through encrypted web protocols. By the time the final hours approached, they had total freedom of movement across the hybrid environment, preparing for the final payload deployment.

### Ending One (Early Success)
By spotting the initial credential stuffing burst and catching the anomalous LSASS handle access early on, your team disrupted the adversary before significant staging occurred. Isolating the compromised workstation and revoking the compromised user session severed the attacker's active connection before their secondary credentials or data exfiltration utilities could take root. The quick containment stopped the breach in its tracks, protecting Meridian Global Freight's shipping manifests and preventing any lateral movement across the network.

### Ending Two (Late Success)
Although the adversary managed to establish persistence, clear event logs, and stage compressed archives locally, your team's system analysis caught the anomaly right as the exfiltration phase kicked off. By spotting the massive asymmetric data transfers routed through commercial cloud storage tools and tracing the unauthorized processes back to the modified account attributes, you successfully severed the command and control channels. While the threat actor accessed local directory contents, your intervention halted the ongoing data egress, contained the credential exposure, and fully remediated the environment before ransomware could drop.

### Ending Three (Failure)
In the final hours of the campaign, with their exfiltration scripts completed and their foothold secured across the domain, the attackers executed their final phase. Ransomware payloads deployed simultaneously across critical on-premises virtualization clusters and cloud storage repositories. File systems froze instantly, text ransom notes dropped across every desktop share, and a public leak site countdown timer went live on the dark web. Meridian Global Freight faced catastrophic operational gridlock, halting international shipping schedules, freezing customs filings, and plunging the logistics firm into an immediate financial and reputational crisis.

