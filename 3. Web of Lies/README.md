# Web of Lies
Failed Roll = Political, Financial, Technological, or Personnel?
Call a Consultant: Pass (Draw a Consultant Card) Fail (-3 penalty on next roll)
Isolation: Pass (3 turns of a +2 bonus to rolls) Fail (Draw an inject)
Crisis Management: Pass (Hint: 1 real, 2 fake) Fail (-2 penalty for 3 turns)

## Scenario
Alert Level: WARNING
What Happened: Unusual User Login (Strange Internet Provider)
Time: 02:14 AM
Account: helpdesk.tier1@BigMoneyCorp.com
Application Accessed: Okta Identity Dashboard

Log Details:
• Current Connection: Consumer Home Internet (Astound Broadband)
• Normal Connection: Corporate Office Fiber (Comcast Business)

Why it flagged: The password and MFA device for this helpdesk account were just changed. While the login is coming from the correct city, the user is connecting from a random home internet provider instead of the dedicated company network they've used for the last two months.

### The Card Set (Attack Path)
- Initial Compromise: Social Engineering
- Lateral Movement: Credential Harvesting
- C2/Exfiltration: Cloud-Based Services as Exfil
- Persistence: New User Added

---

## Card Reveal Details
### Initial Access: Social Engineering
* **The Vishing Log:** A recorded support call shows an inbound request to the helpdesk at 1:45 AM from a caller claiming to be a Tier 1 support technician locked out of their account while setting up a new workstation.
* **The MFA Swap:** Helpdesk ticketing notes confirm that a new mobile device (iPhone 14) was registered to `helpdesk.tier1@BigMoneyCorp.com` during that off-hours call without standard secondary manager approval.
* **The Location Illusion:** The initial successful login immediately following the password reset lists the user's location as Chicago, Illinois, matching the broad geographic area of the real helpdesk employee.
* **The Helpdesk Blindspot:** The real employee log shows they had already clocked out for the night at 5:00 PM and did not log back in, proving the 1:45 AM session was completely unauthorized.
* **The Social Engineering Tell:** Security awareness logs reveal that multiple helpdesk staffers received strange LinkedIn connection requests from a profile posing as a Big Money Corp IT recruiter over the previous 48 hours.

### Pivot/Escalation: Credential Harvesting
* **The OneDrive Footprint:** Audit logs show the compromised `helpdesk.tier1` account performing an unusual bulk search in OneDrive for terms like `password`, `config`, `admin`, and `SSO-Setup`.
* **The Unprotected Backup:** The user account successfully downloads a file titled `Okta_API_Backup_2025.txt` from an abandoned IT deployment folder within the shared company OneDrive directory.
* **The Plaintext Secret:** Forensic analysis of that downloaded document reveals it contained active, hardcoded API keys and high-privilege service account configuration tokens.
* **The Session Hijack:** Immediately after the file download, the attacker uses the harvested keys to query the main single sign-on portal, extracting valid administrative session configuration data.
* **The Administrative Leap:** Within ten minutes, the attacker escalates from a standard helpdesk user account to full, network-wide administrative control across the entire Okta identity directory.

### Persistence: New User Added
* **The Shadow Account:** Azure AD logs flag the creation of a new global user account named `svc-sql-sync` at 3:12 AM, shortly after the attacker began moving through the network.
* **The Privilege Grant:** The newly created shadow account is immediately assigned Domain Admin and Cloud Administrator roles, despite no matching change request ticket existing in Jira.
* **The False Flag:** The account description field is manually filled out to read `Automated SQL Server Backup Sync Account` to trick junior analysts into thinking it belongs to routine system operations.
* **The Utility Implants:** The attacker uses this new shadow admin account to push out lightweight, dual-use remote management tool executables across three high-value internal endpoints.
* **The Local Foothold:** Endpoint logs show these utility implants establishing encrypted outbound connections to external staging servers, ensuring a backup point of entry if the main account is revoked.

### C2/Exfil: Cloud-Based Services as Exfil
* **The Rogue Bucket:** Cloud infrastructure logs reveal a connection from Big Money Corp's internal network to an external, unauthorized AWS S3 bucket named `bmc-data-repository-temp`.
* **The Traffic Blend:** The outbound data flow uses standard, encrypted HTTPS traffic over port 443, making the exfiltration look exactly like legitimate cloud storage synchronization to basic firewall monitors.
* **The Data Staging:** Internal endpoint logs show large batches of sensitive financial records and customer PII being zipped and moved into a temporary local folder right before the outbound transfer begins.
* **The Mass Transfer:** Network monitoring tools record a steady, 45-gigabyte outbound data spike over a two-hour window, directed entirely at that rogue AWS storage endpoint.
* **The Infrastructure Check:** A search of company cloud inventory confirms that this specific AWS account and S3 bucket do not belong to Big Money Corp's official enterprise cloud architecture.

---

## Upping the Ante
### Normal Incident
* **Turn 0: Event (start):** The system baseline has flagged a successful password reset and multi-factor authentication change for an internal account. While the login appears to originate from an expected geographic region, the connection is routing through an unusual consumer home internet provider instead of our dedicated corporate fiber network. You have an active identity anomaly on the board.

* **Turn 2: Event (9 left):** Internal file monitoring systems have picked up quiet, unauthorized navigation inside our corporate cloud repositories. Someone using compromised credentials is deep-diving into old IT backup folders and downloading configuration documentation far outside normal operational boundaries.

* **Turn 4: Event (7 left):** Network perimeter monitors are showing an abrupt, massive spike in outbound cloud traffic. A significant volume of data is actively streaming out of our infrastructure toward an unlisted, external storage endpoint, signaling a major data movement event is underway.

* **Turn 6: Event (5 left):** Central identity logging shows that high-level enterprise administrative privileges are being modified without a matching change request ticket. Concurrently, endpoint monitoring tools are picking up unexpected remote management connections dropping onto workstations across multiple separate departments.

* **Turn 8: Event (3 left):** Endpoint defense alerts are suddenly lighting up across the entire environment. Multiple internal host systems are simultaneously executing automated commands to fetch external files, indicating a synchronized code deployment is attempting to run network-wide.

* **Turn 10: Event (1 left):** The environment has hit total operational failure. Workstation screens are flipping to ransom notes, core servers are locking up, and critical operating systems are crashing into blue screens. This is your absolute final opportunity to trace the footprint and find where this started.



### Live Incident
- **Turn 0: Event (start):** The threat actor uses vishing to compromise the helpdesk and logs in via a consumer proxy ISP. They immediately create a discrete local user account on the initial hijacked workstation to lock in their foothold before poking around.
  * **What to tell the team:** The system baseline detected a successful login that looks technically valid but stands out as highly unusual for this time of night and network path.
    * **Cards Activated:** **Initial Access** and **Persistence** are now active on the board.
    * **Effect:** No Pivot/Escalation or C2/Exfil can be discovered.
    * **Effect:** No Consultant or Isolation can be attempted this turn.


- **Turn 1: Move (10 left):** The attacker attempts to bypass behavioral tracking by timing their commands to match regular shift patterns and mimicking normal employee location data.
  * **Effect:** -2 to next UEBA roll.


- **Turn 2: Event (9 left):** The attacker leverages their secure local foothold to mine the corporate OneDrive, discovering the poorly secured identity provider backup file containing plaintext credentials.
  * **What to tell the team:** There are quiet indicators that someone is navigating internal file repositories looking for something specific, moving past normal operational boundaries.
    * **Cards Active:** **Initial Access**, **Persistence**, and **Pivot/Escalation** can now be discovered.
    * **Effect:** +2 to UEBA rolls this turn due to the unusual OneDrive data mining behavior.
    

- **Turn 3: Move (8 left):** The attacker attempts to disrupt central logging systems by altering event-forwarding rules on compromised local systems or flooding the ingestion pipeline with junk service logs.
  -* **Effect:** -2 to next SIEM Log Analysis roll.


- **Turn 4: Event (7 left):** The attacker pivots into the cloud infrastructure and configures high-volume data staging, triggering a massive outbound synchronization stream toward their rogue, attacker-controlled AWS S3 bucket.
  * **What to tell the team:** External cloud connections are showing a massive shift in activity, signaling that data might be moving places it shouldn't be.
    * **Cards Active:** All four cards (**Initial Access**, **Persistence**, **Pivot/Escalation**, and **C2/Exfil**) are now live and can be discovered on the board.
    * **Effect:** +4 to Firewall Log Analysis rolls this turn due to the massive outbound data spike.
    * **Effect:** +2 to Isolation rolls this turn.
    * **Effect:** No Consultant this turn.
    

- **Turn 5: Move (6 left):** The attacker runs silent Active Directory queries using stealthy parameters specifically designed to identify and map out potential honey-tokens, honey-admins, or fake network shares so they can avoid touching them.
  * **Effect:** -2 to next Active Defense and Cyber Deception rolls.


- **Turn 6: Event (5 left):** The attacker uses the harvested OneDrive credentials to access the main single sign-on portal, granting themselves network-wide authority and deploying utility monitoring implants across scattered internal hosts.
  * **What to tell the team:** High-level administrative configurations are being touched, and anomalous background connections are silently establishing themselves on internal workstations.
    * **Cards Active:** All four cards remain on the board.
    * **Effect:** +2 to SIEM Log Analysis rolls for the remainder of the game.
    * **Effect:** No Crisis Management this turn.
    

- **Turn 7: Move (4 left):** The attacker configures their exfiltration tool to use strict rate-limiting and split-packet data routing, attempting to make the massive AWS data transfer blend in with routine enterprise cloud synchronization traffic.
  * **Effect:** -2 to next Firewall Log Analysis roll.


- **Turn 8: Event (3 left):** The data extraction concludes, and the attacker broadcasts a global execution command to all local utility implants, forcing compromised hosts to run curl commands to download and execute the DragonForce ransomware binary.
  * **What to tell the team:** Endpoint monitoring systems are picking up sudden, unusual system utility requests across multiple departments simultaneously.
    * **Cards Active:** All four cards remain on the board.
    * **Effect:** +4 to Endpoint Analysis rolls this turn.
    * **Effect:** -2 to SIEM Log Analysis rolls this turn.

* **Turn 9: Move (2 left):** The attacker attempts to mask the running ransomware process by renaming the malicious binary to match standard Windows components like `svchost.exe` and nesting the execution thread deep inside trusted system folders to confuse investigators.
  * **Effect:** -2 to next Endpoint Analysis roll.


- **Turn 10: Event (1 left):** Total domain compromise is achieved. Files across the enterprise are fully encrypted, local operating systems begin crashing, and the company receives the triple-extortion ransom demands.
  * **What to tell the team:** Screens are flipping to ransom notes and systems are completely locking up. The incident has reached full operational impact, but executive leadership just authorized an emergency blank check to buy any tool you need. This is your final chance: how do you want to use the funding to make your last stand?
    * **Effect:** The executives have panicked in the final hour and flooded the SOC with emergency budget. The team must choose **one** of two final funding options:
      * **Option 1 (The Targeted Guess):** Narrow the board down to **3 physical cards** (1 right, 2 wrong). The team gets a **+4 modifier** to flip and roll for their single chosen card.
      * **Option 2 (The Panic Spray):** The team selects any **3 detection procedures** from the entire deck and rolls **3 separate dice all at once** (one for each card) at a flat rate to see if any hit.

---

## Full Story
The campaign begins with a highly targeted voice phishing attack against the corporate IT helpdesk. The attackers use open-source intelligence harvested from LinkedIn to impersonate a legitimate system administrator. Armed with personal details and a convincing backstory about a broken corporate phone, the native English-speaking attacker tricks the helpdesk technician into resetting the administrator's password and enrolling a new multi-factor authentication device under the attacker's control.

Immediately following the password reset, the attacker logs into the corporate single sign-on portal. To evade simple geographic blocking, the attacker routes their traffic through a commercial residential proxy network, attempting to blend in with normal domestic internet traffic. However, the proxy IP they land on belongs to a consumer broadband provider entirely out of character for the targeted administrator, who consistently connects via a specific business-class fiber line. This mismatch triggers an identity protection alert for an anomalous sign-on ISP.

Once inside the environment, the attacker moves quietly to avoid endpoint detection. They leverage their initial access to browse corporate collaboration channels and shared file repositories. While searching the team's OneDrive, they locate an unprotected cloud configuration repository containing embedded API keys and high-privilege service account tokens for the corporate identity provider infrastructure.

Using these harvested credentials, the attacker extracts session configuration data straight from the single sign-on portal, granting them broad administrative authority across the network. To establish permanent persistence, they use their elevated rights to create several new user accounts designed to look like routine automated system accounts. They grant these shadow accounts varying levels of administrative privileges, ensuring backup access points if the primary compromised account is locked out. They also drop lightweight remote monitoring and management utility implants on scattered internal systems to maintain local footholds.

With a firm grasp of the entire network topology, the attacker pivots into the cloud infrastructure. They create a legitimate-looking storage bucket inside an attacker-controlled AWS account. Because the outbound traffic flows to standard, trusted cloud endpoints, the data exfiltration blends in with daily business operations. They systematically stage and exfiltrate massive volumes of sensitive corporate data directly into this external bucket.

### Endings
#### Early Success
The team catches the threat actor early in the lifecycle before they can pull off the heist. By jumping on the anomalous identity provider alerts and digging into those quiet cloud repository downloads, the SOC manages to lock down the compromised accounts and terminate the active sessions. Because the team isolates the affected systems and revokes the shadow admin credentials right as the attacker is trying to set up their cloud staging ground, the threat actor is completely cut off. The data stays safe inside the network, the ransomware never gets the chance to deploy, and the attack is stopped dead in its tracks.

#### Eleventh Hour Success
The team takes this right down to the wire, facing full operational chaos. The attacker manages to steal the data and broadcasts the global execution command to deploy DragonForce network-wide. Just as systems start stalling and fans pin to max speed, the SOC utilizes that final surge of emergency executive funding to throw everything at the wall. They accurately trace the ransomware binary back to the initial local persistence footholds and identify the core identity source. While the business still faces a tough cleanup and data liability, the team stops the encryption routine mid-flight, saves the primary domain controllers from total destruction, and prevents the attacker from achieving total domain compromise.

#### Failed Ending
With the data secured for extortion, the attacker prepares for the final phase of the operation. They use their remote utility implants and administrative access to distribute a command across all compromised internal endpoints, forcing them to download the DragonForce ransomware executable from a remote staging server. Within minutes, files across the network are encrypted, and ransom notes populate the screens. The attack achieves total domain compromise, transitioning into a multi-tiered extortion phase involving system lockouts, data leaks, and aggressive harassment.

---

## Sign-off Message
Great Job, Team!

Win or lose, you just went toe-to-toe with tactics inspired by one of the most aggressive threat groups in recent history: **Scattered Spider** (also tracked as UNC3944 or Octo Tempest).

Unlike traditional groups that rely solely on zero-day exploits or complex malware, Scattered Spider dominates through elite social engineering. They're native English speakers who specialize in tricking help desks, executing SIM swaps, and flood-bombing employees with MFA prompts until they click accept. Once inside, they move fast, targeting identity provider settings and cloud environments like AWS and Okta to lock down persistence. They are the group behind the massive disruptions at MGM Resorts and Caesars Entertainment.

If you want to dive deeper into how they operate and see the full breakdown of their technical playbook, check out the official **[CISA Scattered Spider Advisory](https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-320a)**.
