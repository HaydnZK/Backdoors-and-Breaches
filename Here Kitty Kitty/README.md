# Here, Kitty Kitty
**Failed Roll = Political, Financial, Technological, or Personnel?**
- **Call a Consultant**: Pass (Draw a Consultant Card) Fail (-3 penalty on next roll)
- **Isolation**: Pass (3 turns of a +2 bonus to rolls) Fail (Draw an inject)
- **Crisis Management**: Pass (Hint: 1 real, 2 fake) Fail (-2 penalty for 3 turns)
- **Employee Interview**: Get a hint or remove a card 

## Scenario
Alert Type: M365 Defender / Cloud Identity Anomalous Access Rule

Alert Summary: [HIGH] Suspicious Multi-Factor Authentication Bypass & Token Abuse

Alert Details:
Source IP: 185.220.101.42 (Unrated External VPS / Anonymizing Proxy)

Destination: Microsoft 365 / Entra ID Unified SSO Gateway

Protocol: HTTPS (TLS 1.3)

Target Account: `m.sterling@epi-policy.org` (Senior Fellow - Global Energy Transition Policy)

Details: Successful cloud single sign-on authentication event detected from an unknown external IP address bypassing standard geolocation policies. The session established valid OAuth application access tokens immediately after a single sign-in prompt.


## Scenario Cards:
- **Initial Access**: Social Engineering 
- **Pivot/Escalate**: Credential Dumping
- **C2/Exfil**: Cloud Services as Exfil
- **Persistence**: Scheduled Task

---

## Employee Interview Rolls

| D20 Roll | Outcome Category | Result & Narrative |
| --- | --- | --- |
| 1 | Lose Detection | The employee gets extremely defensive and demands HR representation. The ensuing red tape completely blocks your investigation. The Incident Master removes one available detection method from the team roster. |
| 2-5 | Hint 1 | The employee recalls a strange interaction. "I got a LinkedIn message about a security roundtable but had to enter my login credentials on a separate portal to download the agenda." |
| 6 | Draw Starting Condition | Rumors of a cyber incident spread quickly through the office causing panic. Draw a Starting Condition card to represent the operational chaos. |
| 7-9 | Hint 2 | A staff member complains about a weird computer glitch. "My workstation has a brief black command prompt window flash on the screen every time I log in to start my shift." |
| 10 | Draw Consultant | An executive overhears the interview and remembers they have a retainer with a digital forensics firm. Draw a Consultant card to assist the investigation. |
| 11-12 | Hint 3 | An employee mentions an unusual system prompt. "A diagnostic tool popped up yesterday asking for elevated account permissions before it immediately disappeared." |
| 13 | Draw Inject | The interview accidentally tips off a compromised user who might be monitored by the attacker. The adversary accelerates their timeline. Draw an Inject card. |
| 14-15 | Hint 4 | A researcher notes some annoying background activity. "My cloud drive sync icon has been constantly spinning and uploading in the background even when I have not saved anything." |
| 16-19 | Remove Card | The interview provides solid evidence that a specific attack vector was absolutely not used. The Incident Master removes one stale detection card from the board that would not have revealed an active scenario card. |
| 20 | Discover Detection | Critical breakthrough! The employee remembers a crucial detail or hands over a specific file path they noticed earlier. The Incident Master reveals one detection method that will directly uncover an active scenario card. |


---

## Relevant Details
- **Who We Are**: The Internal IT and Security Operations team for Eurasia Policy Institute (EPI), a prominent international policy think tank and research institute specializing in global technology, clean energy policy, and infrastructure resilience.

- **What We Do**: Safeguard sensitive policy drafts, proprietary research datasets, communications between senior fellows and international policy advisors, and cloud identity infrastructure.

- **Workforce & Management Model**: Highly distributed, hybrid organization. Senior fellows, policy analysts, and visiting scholars frequently travel globally, accessing corporate email, messaging platforms, and cloud document repositories remotely from various personal and corporate devices.

- **Identity Management**: Cloud-first identity model utilizing Microsoft 365/Entra ID for Single Sign-On (SSO) across all cloud applications, protected by authenticator-based multi-factor authentication (MFA).

- **Infrastructure Layer**: Cloud-heavy architecture relying heavily on cloud productivity suites (Microsoft 365, Google Workspace), SharePoint/OneDrive research repositories, and a small on-premises local Active Directory domain managing local office workstations.

- **Collaboration & Messaging Protocols**: Heavy reliance on external communication platforms like LinkedIn, WhatsApp, Telegram, and email for reaching out to external journalists, conference organizers, and academic peers.

- **The Environment Baseline**: Steady-state activity includes frequent external document sharing via cloud drives, heavy web browsing, global SSO sign-ins from diverse travel locations, and extensive external email traffic with attachments (PDFs, research briefs, calendar invites).

---

## Card Reveal Details
### Initial Access: Social Engineering

```txt
Received: from mail-out.research-exchange-forum.org (mail-out.research-exchange-forum.org [185.220.101.42])
    by mx.epi-policy.org (MTA) with ESMTPS id 4XyZ9912kL
    for <m.sterling@epi-policy.org>; Thu, 08 Oct 2026 08:14:22 -0400
From: "Dr. Anahita Rezaei" <a.rezaei@research-exchange-forum.org>
Reply-To: "Dr. Anahita Rezaei" <anahita.rezaei.soas@gmail.com>
Subject: Invitation: Global Energy Policy & Clean Infrastructure Summit (Virtual Briefing)
Date: Thu, 08 Oct 2026 08:14:20 -0400

Authentication-Results: mx.epi-policy.org;
    spf=pass (sender IP is 185.220.101.42) smtp.mailfrom=research-exchange-forum.org;
    dkim=pass header.i=@research-exchange-forum.org header.s=s1;
    dmarc=pass (p=none; dis=none) header.from=research-exchange-forum.org
```

* **What we're looking at**: This is a classic lookalike domain attack designed to pass perimeter checks. The attacker registered research-exchange-forum.org and configured valid SPF, DKIM, and DMARC, which is why the mail server marked all three checks as PASS. Notice the Reply-To address points to a generic Gmail account (anahita.rezaei.soas@gmail.com), meaning any response bypasses their domain entirely. It shows how the attacker combined technical validation with social engineering to sneak past our gateway filters.

* **Domain & Delivery Reputation**: The sender domain (research-exchange-forum.org) was registered three weeks ago. The attacker set up valid SPF, DKIM, and DMARC records on their lookalike domain, allowing it to pass gateway checks cleanly.
* **Pre-Contact & Lure**: Log correlation shows the target was previously approached on LinkedIn by a fake persona. After establishing trust, the attacker moved the chat to email and sent a link to a cloud-hosted portal requiring sign-in.


### Pivot & Escalation: Credential Dumping

```txt
EventID: 10
Source: Microsoft-Windows-Sysmon/Operational
Time: Thu, 08 Oct 2026 09:42:15 -0400
RuleName: Credential Access - LSASS Read
UtcTime: 2026-10-08 13:42:15.112
SourceImage: C:\Windows\System32\rundll32.exe
SourceProcessId: 4812
TargetImage: C:\Windows\System32\lsass.exe
TargetProcessId: 672
GrantedAccess: 0x1F0FFF
CallTrace: C:\Windows\SYSTEM32\ntdll.dll+9d324|C:\Windows\System32\KERNELBASE.dll+2d11e|C:\Windows\System32\comsvcs.dll+1a420
```

* **What we're looking at**: This is an endpoint log showing a living-off-the-land credential dumping technique. The attacker executed `rundll32.exe` to leverage the built-in Windows library `comsvcs.dll` and its `MiniDump` function against `lsass.exe`. Notice the `GrantedAccess` code requesting full memory read rights. By abusing a native system binary rather than dropping custom malware like Mimikatz, the attacker attempted to extract hashed credentials directly from system memory while evading basic signature checks.

* **Execution Method**: System logs show an elevated process accessing local security memory via native Windows DLLs to generate a process dump.
* **Persistence Setup**: Shortly after memory access was granted, low-privilege system tasks were registered to maintain user-level access across reboots.


### C2 & Exfiltration: Cloud Services as Exfil

```txt
DateTime: Thu, 08 Oct 2026 11:15:42 -0400
SourceIp: 10.10.45.112 (m.sterling-laptop)
UserAccount: m.sterling@epi-policy.org
UserAgent: Microsoft-OneDrive-Client/26.102.0518
DestinationHostname: api.onedrive.com
RequestPath: /v1.0/drive/special/approot/children/upload
Protocol: HTTPS (TLS 1.3)
BytesSent: 148897280
BytesReceived: 2048
StatusCode: 201 Created
```

* **What we're looking at**: This is a cloud access proxy log showing a living off the cloud exfiltration method. The attacker is leveraging legitimate cloud API endpoints (`api.onedrive.com`) from a compromised user account to upload data. Notice the massive asymmetric transfer with `148897280` bytes sent compared to minimal bytes received. Because `api.onedrive.com` is a trusted enterprise service running over encrypted TLS 1.3, standard perimeter firewalls view this as normal background file syncing rather than malicious data egress.

* **Data Staging & Egress**: Telemetry reveals large outbound HTTPS uploads targeting trusted cloud infrastructure during non-peak usage windows.
* **Access Persistence**: Logs indicate the upload requests leveraged pre-authorized OAuth app tokens and secondary accounts created earlier in the intrusion.


### Persistence: Scheduled Task

```txt
EventID: 4698
Source: Microsoft-Windows-Security-Auditing
Time: Thu, 08 Oct 2026 09:12:04 -0400
TaskName: \Microsoft\Windows\UpdateService\UpdateCheckTask
TaskContent: <Task xmlns="[http://schemas.microsoft.com/windows/2004/02/mit/task](http://schemas.microsoft.com/windows/2004/02/mit/task)"><Triggers><LogonTrigger><Enabled>true</Enabled></LogonTrigger></Triggers><Principals><Principal id="Author"><UserId>S-1-5-21-3891280302-1928374650-10928374-1004</UserId><LogonType>Interactive</LogonType></Principal></Principals><Actions><Exec><Command>powershell.exe</Command><Arguments>-ExecutionPolicy Bypass -WindowStyle Hidden -Enc Y2FsbGJhY2sucHMx</Arguments></Exec></Actions></Task>
SubjectUserName: m.sterling
SubjectDomainName: EPIPOLICY
```

* **What we're looking at**: This is a Security Event log showing the creation of a persistent scheduled task. The attacker named the task `UpdateCheckTask` and placed it in a legitimate-looking system directory to blend in. Notice the `UserId` belongs to a standard domain user rather than `SYSTEM`, meaning it runs entirely in the compromised user context without requiring admin privileges. The trigger is set to `LogonTrigger`, executing a hidden, base64-encoded PowerShell script every time the user logs back in to guarantee their backdoor stays active.

* **Trigger & Execution**: Security event logs record a new automated task configured under standard user privileges that runs hidden scripts upon system logon.
* **Network Activity**: Host telemetry shows the scheduled task periodically reaching out to external web endpoints to verify session connectivity.


---

## Heat Raisers
- **Turn 2: The Help Desk Ping**
A few employees submit casual tickets mentioning odd follow-up messages on LinkedIn and email from an external speaker who presented at a recent industry webinar. The speaker seems eager to share follow-up research materials, but some staff members are questioning if the sender's account was compromised or spoofed.

- **Turn 4: The Anomalous Sign-In**
The SOC receives a low-severity alert regarding a successful single sign-on event for a corporate account originating from an unfamiliar ISP. The user insists they are working from home as usual, but logs show two active sessions registered from different geographical regions within minutes of each other.

- **Turn 6: The VIP Inquiry**
The executive team reaches out to IT Security after a high-ranking director reports that several external partners received emails sent directly from her mailbox asking for urgent reviews of confidential strategy documents. The director swears she never sent those messages.

- **Turn 8: The Bandwidth Spike**
Network operations notices a subtle, sustained spike in outbound network traffic hitting external infrastructure during non-business hours. The traffic is fully encrypted and originating from standard user subnets, making it blend in with routine web browsing at first glance.

- **Turn 10: The Cloud Lockout (The Finale)**
The incident escalates to a full-blown crisis. Executive staff and system administrators suddenly lose access to core corporate cloud portals and email tenants. The threat actor has taken over administrative identity controls, revoked user sessions across the entire company, and issued an extortion demand claiming proprietary research and sensitive internal data have been compromised.


---

## Final Story
Three weeks before the incident, threat actors established a lookalike domain named research-exchange-forum.org, carefully setting up valid SPF, DKIM, and DMARC records to pass default perimeter security checks. Using a fake LinkedIn persona under the name Dr. Anahita Rezaei, the adversary built rapport with Senior Fellow M. Sterling over several days. Once trust was established, the attacker sent an email invitation to a virtual briefing titled the Global Energy Policy & Clean Infrastructure Summit. The email directed Sterling to an attacker-controlled credential harvesting portal designed to mirror the Eurasia Policy Institute's Entra ID login page. When Sterling logged in, the portal relayed their credentials and multi-factor authentication prompt in real time, granting the attacker valid OAuth tokens and an active session.

With initial access secured in Sterling's M365 account, the attacker leveraged their cloud access to drop a light initial payload onto Sterling's local workstation. To ensure they maintained a foothold even if password resets or token revocations occurred, the adversary focused on stealthy local persistence. They created a hidden scheduled task named UpdateCheckTask inside the legitimate-looking \Microsoft\Windows\UpdateService\ system directory. Configured under standard user privileges with a LogonTrigger, this task executed a base64-encoded PowerShell script every time Sterling logged into the computer, quietly establishing an outbound C2 beacon without raising administrative alarms.

To expand their reach across the institute's internal network and search for elevated permissions, the attacker sought local credential material. Instead of downloading custom malware that might trigger endpoint alerts, they turned to Living-off-the-Land techniques using native Windows binaries. Executing system utilities, the attacker invoked rundll32.exe to call the MiniDump export function within comsvcs.dll directly against the lsass.exe process. This dumped the Local Security Authority Subsystem Service memory into a temporary local file, allowing them to extract cached credential hashes from memory while bypassing standard antivirus signature checks.

With unrestricted access to Sterling's workstation and cloud storage, the adversary targeted sensitive policy drafts, proprietary research datasets, and upcoming international briefing notes. Rather than routing stolen data to a suspicious external IP address, the attacker staged the collected files into compressed archives and abused Sterling's legitimate enterprise OneDrive client. They uploaded the archives directly to cloud storage endpoints via api.onedrive.com. Because the transfer occurred over encrypted TLS 1.3 to a trusted enterprise cloud service, standard perimeter firewalls viewed the massive outbound upload as routine background file synchronization.


### Ending One (Early Success)
By identifying the threat early in the lifecycle, your team disrupted the adversary before significant data exposure occurred. Recognizing the initial MFA bypass and isolating Sterling's account severed the attacker's active session before their persistent scheduled task could establish a permanent foothold. The quick containment contained the breach to a single compromised endpoint, successfully protecting the institute's sensitive policy drafts and preventing lateral movement across the internal network.


### Ending Two (Late Success)
Although the adversary managed to establish persistence and access local LSASS memory, your team's systematic log analysis caught the anomaly during the exfiltration phase. By spotting the massive, asymmetric OneDrive API transfers and tracing the activity back to the hidden scheduled task, you successfully severed the attacker's command and control channel. While the threat actor accessed some local research files, your delayed intervention effectively halted the ongoing data egress, contained the credential exposure, and fully remediated the compromised assets.


### Ending Three (Failure)
In the final hours of the campaign, with their exfiltration script completing its final archive upload to OneDrive, the attacker executed a batch cleanup script that wiped event logs, encrypted the victim workstation's master boot record, and systematically burned their foothold. The sudden host crash triggered an emergency outage, but by then it was far too late. The stolen policy briefs and unreleased global energy research datasets were leaked onto a dark web forum, triggering an immediate crisis: partner organizations pulled funding, international media broke stories on the institute's compromised research integrity, and the Eurasia Policy Institute faced severe regulatory fines and reputational ruin.

