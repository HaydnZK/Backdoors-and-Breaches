# The Trickle Effect
Failed Roll = Political, Financial, Technological, or Personnel?
Call a Consultant: Pass (Draw a Consultant Card) Fail (-3 penalty on next roll)
Isolation: Pass (3 turns of a +2 bonus to rolls) Fail (Draw an inject)
Crisis Management: Pass (Hint: 1 real, 2 fake) Fail (-2 penalty for 3 turns)

## Scenario
Ticket ID: #Gov-8821
Priority: Medium (Escalated)
From: Help Desk Tier 1
Subject: Recurring "Sluggishness" on Multiple Workstations

Description:
"Over the last 48 hours, we've received about a dozen tickets from the Accounting and HR departments complaining that their systems are 'crawling.' Users report that Excel takes minutes to open and their laptop fans are running extremely loud, even when no apps are open.

We initially thought it was a bad Windows Update, but we've cleared the cache and rebooted several machines, and the high CPU usage returns within twenty minutes. We also noticed some weird 'service' names in Task Manager that we didn't recognize, but they disappear when we try to right-click them.

Escalating to Security for a look-see. It's probably just a weird bug, but Accounting is getting frustrated because they can't get payroll finished."

### The Card Set (Attack Path)
- Initial Compromise: Phishing
- Lateral Movement: SMB Weakness
- C2/Exfiltration: DNS Tunneling
- Persistence: Malicious Service

---

## Card Reveal Details
### Initial Compromise: Phishing
- A user in the HR department received an email titled "RE: Mandatory 2026 Benefits Adjustment" from a lookalike domain gov-benefits.org.
- The email contained a password-protected ZIP file; the password was provided in the body of the email to bypass automated sandbox scanning.
- Inside the ZIP was a VBScript file disguised with a PDF icon named Benefits_Summary_v4.pdf.vbs.
- Mail logs show the attachment was opened by three different employees across two departments.
- Shortly after the file was opened, a process named wscript.exe was seen making an outbound connection to a known malicious IP.

#### Clarifying Context
It's a classic TrickBot entry point. An employee was tricked into running a malicious script by an email that looked like it came from an official government source. This initial callback is what gave the attacker the foothold they needed to begin pivoting toward the cloud and internal servers.

### Lateral Movement: SMB Weakness (EternalBlue/MS17-010)
- Network logs show a sudden spike in traffic on port 445 (SMB) originating from the HR workstations.
- The traffic consists of malformed packets typical of the EternalBlue exploit, targeting unpatched legacy systems in the Finance VLAN.
- Several older file servers that were exempt from recent patching cycles have suddenly gone offline or rebooted.
- The lsass.exe process (which handles credentials) was remotely injected with malicious code on multiple machines.
- Authentication logs show a service account being used to log into dozens of machines simultaneously across the organization.

### C2/Exfiltration: DNS Tunneling
- The firewall is logging an unusually high volume of DNS queries to a series of randomized subdomains (e.g., mx1.a7b9.cc, ns4.z2p8.top).
- The queries are much larger than a standard DNS request, often reaching the maximum character limit.
- Traffic analysis shows these requests are being sent at a steady heartbeat interval, even when no users are logged in.
- Traditional web filters didn't catch this because the traffic isn't using HTTP/HTTPS; it's hiding inside the name-lookups that the network uses every day.
- Total outbound data volume over the DNS port has reached several gigabytes, suggesting sensitive files are being chunked and sent out.

#### Clarifying Context
The attacker is using DNS as a secret tunnel. It's hard to block because if you turn off DNS, the whole internet stops working for the organization. This constant outbound chatter is what the DLP started picking up on once the volumes got high enough to trigger the alert.

### Persistence: Malicious Service
- A new Windows Service has been created named "Windows Remote Management Core" (not to be confused with the legitimate WinRM).
- The service is pointing to a hidden executable located in C:\Windows\System32\config\systemprofile\AppData\.
- This service is configured to run under the LocalSystem account, giving it full control over the host.
- Forensic analysis of the registry shows the service is set to "Delayed Auto-Start," making it harder to spot during a fast boot.
- If the service process is killed, a "Recovery" trigger immediately restarts it and generates a new random filename to evade signature-based antivirus.

#### Clarifying Context
Even if the network is cleaned, this service ensures the malware "wakes up" every time the computer is turned on, allowing the attackers to get back in. This was established early on to make sure that even if we cut the C2 connection, the infection could persist and wait for new instructions.

---

## Upping the Ante
### Normal Level: Incident Heat Raisers
#### Turn 7: The DDoS Discovery (DDoS Event)
- "Your organization's IP range is currently the primary source of a massive Layer 7 DDoS attack hitting a major financial institution. The traffic is originating from your internal server VLAN."

#### Turn 5: The Data Drain (DLP Event)
- "The DLP is reporting high volumes of data leaving the network. The pipe is saturated with outbound traffic as sensitive files are being chunked and sent to randomized subdomains."

#### Turn 3: The Evidence Burn (Logic Bomb Event)
- "A logic bomb has gone off and is clearing evidence of the malware. Forensic artifacts like Event Logs, Prefetch, and Shimcache are being purged in real-time across the network."

#### Turn 1: The Final Goodbye (Wiper Event)
- "Several systems have just gone offline. Remote technicians report seeing 'Missing Operating System' errors on the screens as the malware wipes the Master Boot Record."

### Live Incident
- Turn 0: Event (start): The threat actor gains initial entry and pivots through the network before dropping cryptominers.
  - Effect: No C2/Exfil or Persistence until turn 2. 

- Turn 1: Move (10 left): The threat actor attempts to modify the cloud storage retention policy to immediate delete for the current session.
  - Effect: -2 to Cloud Event Log Analysis this turn.

- Turn 2: Event (9 left): The malware establishes connection to the threat actor's C2 and gains instructions to create the Malicious Service to gain persistence. 
  - Effect: C2/Exfil and Persistence can be discovered. 
  - Effect: +2 to rolls that detect C2/Exfil and Persistence this turn. 

- Turn 3: Move (8 left): The threat actor attempts to quietly clear the Firewall Logs.
  - Effect: -2 to Firewall Log Analysis this turn. 

- Turn 4: Event (7 left): The threat actor uses the zombies to perform a DDoS
  - Effect: +4 to rolls that detect C2/Exfil this turn. 
  - Effect: +2 to Isolation rolls this turn. 
  - Effect: No consultant this turn. 

- Turn 5: Move (6 left): The threat actor attempts to flood the SIEM with thousands of junk decoy alerts.
  - Effect: -2 to SIEM Log Analysis this turn.

- Turn 6: Event (5 left): The DLP is reporting high volumes of data leaving the network.
  - Effect: +2 to rolls detecting C2/Exfil for the remainder of the game. 
  - Effect: No Crisis Management this turn. 

- Turn 7: Move (4 left): The threat actor uses compromised credentials to perform normal administrative tasks to blend in with legitimate traffic.
  - Effect: -2 to UEBA this turn.

- Turn 8: Event (3 left): A logic bomb has gone off and is clearing evidence of the malware. 
  - Effect: +2 to UEBA rolls this turn.
  - Effect: -2 to Memory & Endpoint Analysis rolls this turn. 

- Turn 9: Move (2 left): The threat actor attempts to stop the logging service and wipe the temporary cache on the primary domain controller.
  - Effect: -2 to Server Analysis this turn.

- Turn 10: Event (1 left): Users are reporting an influx of "missing operating system" errors. 
  - Effect: +2 to rolls discovering Persistence this turn.

---

## Full Incident Story
The incident started with the HR department being the target of a sophisticated phishing campaign during a period of high-volume internal communication due to an upcoming open enrollment and change in company-provided health insurance providers. An employee opened a password-protected ZIP archive and executed a hidden script, which served as a loader for the malware. This immediately established a foothold and began calling out to the attackers for instructions.

Once the foothold was secure, the attacker began pivoting toward the cloud infrastructure, attempting to modify log retention policies to hide their initial tracks. Back on the local network, the malware utilized its modular nature to scan for weaknesses. It identified multiple legacy servers in the Finance VLAN vulnerable to EternalBlue. Because these systems hadn't been patched due to software compatibility concerns, the malware was able to worm through the network unimpeded, harvesting administrative credentials along the way.

As the botnet grew, the attacker established a DNS tunnel for C2 to avoid detection by standard web filters. By hiding instructions within legitimate-looking DNS queries, the attacker maintained consistent contact and began the first phase of the operation: deploying cryptominers to take advantage of the agency's idle CPU power.

As the investigation progressed, the attacker sold the network's bandwidth to a third party to launch a massive DDoS attack against a major financial institution. This loud diversion was intended to distract defenders while the attackers moved toward their true objective. During this chaos, the DLP started flagging high volumes of data leaving the network as the attackers began their final exfiltration of sensitive files.

Before the network could be fully isolated, the attacker established persistence through a Malicious Service on multiple core servers. Disguised as a system utility, this service ensured the malware would stay embedded in the environment. In the final act, the attackers attempted to burn the bridge by deploying a logic bomb to scrub forensic evidence and a wiper module to destroy master boot records, leaving the organization in a state of operational failure with no logs to show what was stolen.

### Successful Endings
#### **Before Bricked Servers**
By neutralizing the threat before the wiper could trigger, the team successfully cut the C2 connection just as the attackers were preparing to pivot from the DDoS distraction to full-scale network destruction. While the agency is currently dealing with the fallout of the DDoS and the cleanup of the cryptominers, your rapid response prevented the logic bomb and wiper protocols from ever triggering. The domain controllers remain intact, and while some data was staged for exfiltration, you saved the organization from a total forensic wipe and months of infrastructure rebuilding.

#### **After Bricked Servers**
The team managed to regain control just as the logic bomb was beginning to take hold. Several core servers and domain controllers were already knocked offline by the initial wiper module, and the organization faces a significant recovery effort to restore those systems from backups. However, by identifying the persistence mechanisms before the final turn, you stopped the total erasure of event logs and forensic artifacts. This photo finish means that while the damage is severe, the security team still has the evidence needed to track exactly what was stolen, preventing a complete operational blackout.

### Failed Ending
The attacker successfully completed their mission. While the immediate DDoS was eventually mitigated by external providers, the threat actor maintained their foothold long enough to exfiltrate sensitive HR and financial records. The logic bomb successfully scrubbed the forensic trail, and the wiper module has left the organization in a state of complete collapse. With the master boot records destroyed and no logs left to analyze, the agency is left in the dark about what was stolen and faces a total rebuild of its digital infrastructure.
