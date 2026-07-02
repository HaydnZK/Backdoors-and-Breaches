# Salty Storm
Failed Roll = Political, Financial, Technological, or Personnel?
Call a Consultant: Pass (Draw a Consultant Card) Fail (-3 penalty on next roll)
Isolation: Pass (3 turns of a +2 bonus to rolls) Fail (Draw an inject)
Crisis Management: Pass (Hint: 1 real, 2 fake) Fail (-2 penalty for 3 turns)

## Scenario
**SIEM Alert Trigger:** Active Directory Modification Detected
**Severity:** HIGH
**Timestamp:** 2026-06-23T01:42:11Z
**Alert ID:** ALRT-4402-GPO

**Alert Description:**
An automated monitoring rule triggered due to an unapproved change to a critical Group Policy Object. A directory service object modification event occurred on the primary domain controller, altering active security configurations.

**Log Details:**
- **Target Server:** DC-PROD-01
- **Event Source:** Microsoft-Windows-Security-Auditing
- **Event ID:** 5136
- **Object Class:** groupPolicyContainer
- **Object DN:** CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=Policies,CN=System,DC=SaltyStorm,DC=local
- **Attribute Name:** gPCFileSysPath
- **Attribute Value:** \SaltyStorm.local\sysvol\SaltyStorm.local\Policies{31B2F340-016D-11D2-945F-00C04FB984F9}\Machine\Scripts\Startup
- **Subject Account:** Vendor-NetOps-Admin

**Analyst Notes:**
The account associated with this modification is a third party provider. No corresponding authorization change ticket is scheduled in the inventory system for this maintenance window. Immediate verification of the script content inside the specified Sysvol path is required to confirm authorization status.

## Scenario Cards
- **Initial Access**: ???
- **Pivot/Escaltion**: ???
- **C2/Exfiltration**: ???
- **Persistence**: ???

---

## Relevant Details
* **Who We Are:** SaltyStorm Communications is a major regional internet service provider (ISP) and telecommunications infrastructure operator.
* **What We Do:** We manage critical regional routing hubs, core network backbones, and cell towers, handling high-volume subscriber traffic and routing data packets across the state.
* **Workforce & Management Model:** Our main engineering teams operate out of local network operations centers (NOCs), but we rely heavily on trusted, third-party managed service providers and network vendors who possess broad administrative access to our infrastructure.
* **Identity Management:** We utilize a traditional, on-premises Active Directory environment (SaltyStorm.local) to manage administrative privileges, authentication servers, and access rights across all core systems and domain controllers.
* **Infrastructure Layer:** The infrastructure is a hybrid ecosystem, relying heavily on physical, high-performance edge routing appliances, BGP switches, and local directory servers alongside a smaller cloud footprint used for customer billing portals.
* **Vendor Access Protocols:** Our external NetOps vendors connect through a highly privileged, dedicated virtual private network (VPN) segment designed to let them push configuration updates to core network switches during approved change windows.
* **The Environment Baseline:** Our operations rely on strict change management tickets. Any modification to our Active Directory structure or core group policy objects outside of an approved maintenance window immediately signals an operational anomaly.

---

## Card Reveal Details
### Infected Authorized Vendor Laptop (Initial Access)
* **Compromised Supply Chain:** The threat actor gained initial entry by targeting a trusted third-party vendor with direct remote access privileges to our internal network environment.
* **Stolen Remote Access Credentials:** Rather than brute-forcing the perimeter, the attacker harvested legitimate VPN and out-of-band management credentials directly from the vendor's local configuration files.
* **Malicious DLL Sideloading:** Forensics on the vendor asset revealed that the attacker utilized DLL sideloading against legitimate, trusted software packages on the laptop to execute their initial payloads without alerting security tools.
* **Session Hijacking:** The attacker actively hijacked established administrative sessions originating from this laptop, allowing them to ride along inside our corporate fiber network completely unhindered.
* **Geographic Anomaly:** While the login session matched the provider's correct corporate city, deeper inspection showed the traffic was routed through a strange consumer home internet connection instead of the vendor's dedicated commercial backbone.

#### **Bonus: Vendor Portal (Alternative Initial Access)**
* **Exposed Admin Interface**: A perimeter scan revealed that a legacy external-facing vendor management portal, originally used by legacy engineering teams, was left exposed to the public internet without proper firewall filtering.
* **Lack of Multi-Factor Authentication**: Unlike our primary corporate gateways, this specific partner interface lacked multi-factor authentication requirements, relying solely on single-factor static credentials.
* **Brute-Force Spraying Campaign**: Central authentication logs showed a highly targeted password spraying campaign targeting common service accounts, which successfully identified a single valid credential pair.
* **Session Cookie Theft**: Deeper log analysis indicated that the attacker successfully imported valid session tokens stolen from an active session, bypassing traditional geolocation blocks.
* **Exploitation of Known Vulnerability**: The threat actor took advantage of an unpatched administrative bypass flaw within the portal framework to directly elevate their privileges to a system administrator level before pivoting inside the local subnet.

### GPO Modification (Persistence)
* **Directory Services Object Event:** An automated file integrity monitor flagged a high-severity Event ID 5136 on a primary domain controller, indicating an unauthorized modification inside the Active Directory structural layer.
* **Startup Script Insertion:** The attacker modified the `gPCFileSysPath` attribute within a core group policy container, altering a system startup parameter to point to a hidden path.
* **Sysvol Path Poisoning:** Investigation of the standard Sysvol directory revealed a malicious script hidden deep inside the policies folder, configured to run with local system privileges every time the server boots.
* **Security Control Bypassing:** The altered policy was specifically crafted to quietly weaken local security configurations, clear event-forwarding rules, and disable event auditing across specific subnets.
* **Zero Change Ticket Matching:** Cross-referencing the timestamp of the Active Directory change against our inventory system confirmed there were no authorized maintenance windows or active change management tickets open for that time block.

### Golden Ticket Attack (Pivot/Escalate)
* **KRBTGT Account Compromise:** The threat actor successfully dumped the password hash of the critical KRBTGT service account from a local domain controller memory space.
* **Forged Kerberos Tickets:** With the KRBTGT hash in hand, the attacker generated completely custom, self-signed Kerberos Ticket Granting Tickets (TGTs) that bypass standard authentication checks.
* **Arbitrary Group Membership:** The forged tickets were injected with fake security identifiers (SIDs), falsely granting the attacker membership to high-privilege groups like Enterprise Admins and Domain Admins.
* **Massive Ticket Lifetimes:** Network packet reviews caught Kerberos tickets with highly unusual, extended lifetimes set for up to ten years, allowing the attacker to bypass normal daily authentication challenges.
* **Pass-the-Ticket (PtT) Execution:** Responders identified memory anomalies where these forged tickets were directly injected into active system memory sessions to move laterally across internal infrastructure without providing actual passwords.

### Bridged System (C2/Exfil)
* **Covert Tunneling Protocols:** The threat actor modified network boundaries by spinning up unauthorized Generic Routing Encapsulation (GRE) and IPsec protocol tunnels over non-standard high ports.
* **Evasion of Security Inspection:** By creating these raw, low-level network bridges, the attacker routed their outbound communications completely around our primary firewall inspection and web proxy checkpoints.
* **Dual-Channel Infrastructure:** Network telemetry showed the attacker utilized a hybrid command and control layout, combining anonymous Virtual Private Servers (VPS) with trusted public cloud platforms to hide their data transfers.
* **Living-off-the-Land Network Tools:** Instead of dropping known hacking software, the actor leveraged native Windows administrative binaries and built-in network commands to bridge different network segments together.
* **Data Staging Areas:** Forensics uncovered hidden staging directories on the bridged machine where large sets of sensitive internal database files had been collected, compressed, and encrypted prior to external transmission.

---

## Heat Raisers
### Normal Incident
Turn 2: The Infrastructure Blip
The Network Operations Center reports a brief routing blip. A core edge router on the regional management plane suddenly dropped its primary BGP neighbor relationship for forty five seconds before recovering. Central logs show this glitch occurred right after an administrative configuration change was pushed from the vendor VPN segment.

Turn 6: The Secondary Foothold
An automated internal network sweep picks up an unapproved, active network interface on a critical database server. The system logs show that a secondary, dual homed bridge has been activated on the host, and it is quietly establishing an outbound connection that completely bypasses the main corporate firewall proxy.

Turn 8: The Telemetry Blackout
Central monitoring loses visibility into a major regional routing hub. Security event forwarding and netflow collection services are suddenly stopping across multiple core network appliances simultaneously, indicating that someone is systematically disabling the logging features on compromised infrastructure.

Turn 10: Total Domain Compromise (The Finale)
The incident hits full operational impact. The attacker utilizes their forged Golden Ticket credentials to push out global configuration updates across the entire Active Directory database, effectively locking out your internal administration team. The threat actor now has complete, permanent control over the ISP routing backbone, allowing them to intercept or redirect any subscriber traffic passing through the state.

### Live Incident
* Attacker Turn 0 (10 left): Hidden Event: The threat actor has achieved initial access via a public facing edge device and has already established a persistence mechanism to maintain long term remote entry.
  - Effect: Defenders can discover the Initial Access and Persistence cards. 

* Attacker Turn 1 (9 left): Move: The attacker runs specialized clean-up scripts to selectively delete Windows Security events, clear local router command history logs, and systematically disable event-forwarding agents across core network switches.
  - Effect: -2 to SIEM Log Analysis

* Attacker Turn 2 (8 left): Hidden Event: The threat actor is executing lateral movement across the internal management plane while simultaneously staging and initiating data exfiltration to external servers.
  - Effect: Defenders can discover the Pivot/Escalate and C2/Exfil cards. 

* Attacker Turn 3 (7 left): Move: The threat actor deploys a sophisticated, stealthy kernel-mode rootkit (such as Demodex) to intercept system API calls and hide their malicious processes, files, and open network connections directly from the local operating system.
  - Effect: -2 to Endpoint Analysis

* Attacker Turn 4 (6 left): Event: The infrastructure team reports that the network has become completely fragmented. The attacker has modified core routing tables and established unauthorized logical bridges across multiple segments. Any automated or manual attempts to segment network traffic or wall off compromised zones are failing completely because the traffic is automatically rerouting through rogue paths.
  - Effect: -4 to Isolation

* Attacker Turn 5 (5 left): Move: To blend in seamlessly with everyday network administration, the attacker completely avoids noisy commercial hacking tools, relying instead exclusively on native Windows administrative binaries (Living-off-the-Land) and legitimate system commands during off-hours.
  - Effect: -2 to UEBA

* Attacker Turn 6 (4 left): Event: As the situation escalates, internal IT attempts to provision external access for your outside forensics partner. However, the attacker has completely hijacked the out-of-band management plane and revoked external VPN gateway access keys. Outside connections are being dropped at the perimeter, effectively locking your external experts out of the environment.
  - Effect: -4 to Call a Consultant

* Attacker Turn 7 (3 left): Move: The attacker alters network infrastructure boundaries by spinning up covert Generic Routing Encapsulation (GRE) and IPsec protocol tunnels over non-standard high ports, effectively routing all their staging traffic completely around corporate inspection points.
  - Effect: -2 to Firewall Log Review

* Attacker Turn 8 (2 left): Event: Chaos hits the executive level. The threat actor has gained access to internal communication directories and is actively disrupting corporate messaging profiles. Rogue messages are being sent from executive accounts, and the internal telemetry readouts being fed to leadership are completely contradictory, throwing the entire leadership team into total disarray.
  - Effect: -4 to Crisis Management 

* Attacker Turn 9 (1 left): 
- Move: The threat actor successfully compromises the internal TACACS+ or RADIUS authentication servers, modifying the configurations to intercept administrative credentials and bypass access controls without triggering local security failures.
  - Effect: -2 to Server Analysis
- Event: Our internal network engineering team just pulled off a massive breakthrough. By combing through raw physical layer connections, they successfully located a rogue administrative console that the threat actor left exposed. We have a temporary, high-visibility window into their operations before they notice and kill the connection. Management is giving the Incident Response team full authority to leverage this window in one of two ways. You must choose right now:
  - Option 1: The network engineers use the console to trace a single, high-confidence signal. I will give you three specific cards: two are false leads, and one is the exact card you need. You get one single roll to guess the right card, but you get a massive +5 modifier to that roll due to the clean signal data.
  - Option 2: The engineers open up the telemetry pipe as wide as possible for a brief, frantic burst. You get to choose any three cards on the board with zero hints from me. You then get three rolls to distribute across those three cards for a high-speed lightning round, but you get no bonuses to the rolls.

* Attacker Turn 10 (0 left): Event: Total domain compromise is achieved. The attacker uses forged administrative credentials to modify the entire identity directory, locking out internal IT staff and seizing permanent control of the ISP routing backbone.

---

## Final Story
The intrusion began quietly when Salt Typhoon exploited a critical vulnerability on our public facing edge infrastructure to gain initial access. The moment they got inside, they immediately set up shop, dropping backdoors and altering core router configurations to establish a rock-solid persistence mechanism before a single alarm could ring. To cover their tracks right out of the gate, they executed automated clean-up scripts that selectively wiped Windows Security events and disabled event-forwarding agents across our core network switches, completely blinding our SIEM log analysis.

With their foothold secure, the threat actor began lateral movement across our internal management plane, escalating their privileges and quietly staging subscriber database files for exfiltration. Our team fought back using endpoint analysis, but the attackers deployed a stealthy kernel-mode rootkit to intercept system API calls, successfully hiding their rogue processes, malicious files, and open network connections directly from our local operating system detection tools.

The attack escalated into an absolute nightmare when the threat actor altered core routing tables, fragmenting our network and establishing unauthorized logical bridges that completely broke our isolation procedures. Any manual or automated attempts by our team to segment the network or wall off compromised zones failed instantly as the traffic simply rerouted through their rogue paths. They blended in perfectly with normal day-to-day operations by using living-off-the-land techniques, running native administrative tools during off-hours to bypass our anomaly detection models.

When we tried to bring in outside consultants to help salvage the network, we found our out-of-band management plane hijacked and our external VPN gateway access keys completely revoked. Our partner forensic experts were left totally locked out at the perimeter. To make matters worse, the attackers bypassed our firewall log reviews entirely by spinning up covert GRE and IPsec tunnels over non-standard high ports to sneak their final staging data out of our environment.

In the final hours, absolute chaos hit the executive level. The threat actor gained access to our internal communication directories, hijacking corporate messaging profiles to send rogue administrative messages and feeding completely contradictory telemetry data to leadership, throwing our entire crisis management strategy into total disarray. In a final, desperate move to secure total control, Salt Typhoon compromised our internal TACACS+ authentication servers to intercept admin credentials, successfully bypassing access controls without triggering local security failures.

Just as the clock was running out, our internal network engineering team managed to pull off a massive breakthrough. By combing through raw physical layer connections, they located a rogue, exposed administrative console that the threat actor accidentally left open. This granted our incident responders a temporary, high-visibility window of raw telemetry data before the attackers noticed the leak and killed the connection.

### Ending 1: Victory (The Successful Eviction)
Our responders leveraged this critical, temporary window of telemetry data perfectly. Using the high-confidence signal data, the team successfully identified the primary compromised core node and purged the threat actor's unauthorized SSH keys, malicious group policy objects, and hidden rootkits before the attackers could close the door. By evicting the backdoors, rotating our compromised administrative credentials, and locking down our authentication directories, we successfully severed Salt Typhoon's connection, stabilized the ISP backbone, and saved the network from total compromise.

### Ending 2: Failure (Total Domain Compromise)
Unfortunately, the telemetry data was too fragmented, and the brief window vanished before our team could pinpoint the threat actor's primary persistence mechanisms or locate the hijacked accounts. With our defenses blinded, our outside experts locked out, and our internal coordination broken, Salt Typhoon seized the opportunity. The attackers used their forged administrative credentials to completely modify the entire Active Directory identity directory. Internal IT staff were permanently locked out of their own systems, and the threat actor seized absolute, permanent control of our ISP routing backbone. The entire corporate infrastructure fell into enemy hands, resulting in a catastrophic, total domain compromise.

### Ending 3: Midgame Victory (Early Containment)
Our incident response team refused to let the threat actor gain a solid foothold. The moment Salt Typhoon established initial access and attempted to plant their backdoors, our analysts caught the subtle configuration anomalies on the edge infrastructure. Even though the attackers scrambled to run their clean-up scripts and wipe our SIEM logs, our team had already captured the crucial telemetry.

By executing aggressive endpoint analysis and checking our primary connection paths, we caught their lateral movement right as it began. We identified the compromised administrative accounts and the staging areas before the attackers could deploy their rootkits or fragment our routing tables.

Management immediately authorized an emergency credentials rotation across all core network switches and local systems, completely killing the threat actor's active sessions. By isolating the exposed edge devices and locking down our authentication gateways early, we successfully evicted Salt Typhoon before they could exfiltrate data, hijack our communication channels, or disrupt our infrastructure. The backbone remained secure, and the network was fully stabilized without requiring outside intervention or crisis management.

---

## Sign-off Message
Great job, team!

Win or lose, you just went head-to-head with tactics inspired by one of the most sophisticated cyber espionage threats out there today: **Salt Typhoon** (also tracked as GhostEmperor, FamousSparrow, or Earth Estries).

Unlike traditional groups that use flashy ransomware to demand a fast payout, Salt Typhoon is a state-sponsored actor that plays the long game. Their primary objective is quiet, pervasive intelligence gathering. They made massive waves across the industry by successfully infiltrating the core routing infrastructure of major global telecommunications companies and internet service providers.

By taking over the hardware that manages internet backbone traffic, they can monitor sensitive data streams, map network routes, and intercept communications on a massive scale without needing to target individual users directly. They are highly adept at evading standard security tools by using kernel-mode rootkits to mask their files, hiding inside legitimate networking services, and blending in using native administrative binaries.

If you want to look under the hood and see the full breakdown of their technical playground, you can check out my deep-dive research repository on the **[GitHub Salt Typhoon CTI Page](https://github.com/HaydnZK/Research-and-Methodology/tree/main/Salt_Typhoon_CTI)**.

To study the technical defense blueprints and mitigation rules recommended by federal agencies, feel free to dive into the official **[CISA China Cyber Actor Advisories](https://www.cisa.gov/topics/cyber-threats-and-advisories/nation-state-cyber-actors/china)**.
