# Ground Shaker
Failed Roll = Political, Financial, Technological, or Personnel?
Call a Consultant: Pass (Draw a Consultant Card) Fail (-3 penalty on next roll)
Isolation: Pass (3 turns of a +2 bonus to rolls) Fail (Draw an inject)
Crisis Management: Pass (Hint: 1 real, 2 fake) Fail (-2 penalty for 3 turns)

## Scenario
Alert Type: Central SIEM High-Severity Rule

Alert Summary: [ALERT] Unauthorized Out-of-Band SSH Connection to Core Edge Hardware

Alert Details:
Source: 10.4.12.88 (Workstation - Marketing Desk 4)
Destination: 10.0.0.1 (Core Edge Router - edge-rtr-01)
Protocol: SSH (Port 22)
User Account: netadmin_svc

Details: High-privilege administrative SSH session initiated to core routing infrastructure from an unauthorized, non-jump-box IP address.


Description:
Basically, an admin-level remote control connection was opened to our core router, but it came from a random marketing computer instead of our secure IT management servers.

Even though a valid admin account was used, someone accessing our core network gear from an unapproved desk computer is a huge red flag. It usually means an attacker either stole admin credentials or compromised that workstation to secretly backdoor the router.


## Scenario Cards
- **Initial Access**: ???
- **Pivot/Escaltion**: ???
- **C2/Exfiltration**: ???
- **Persistence**: ???

---

## Relevant Details
* **Who We Are:** Enterprise Network Operations Center (NOC) acting as first-line operational defenders, managing core routing infrastructure, edge gateways, and internal switching layers for a multi-site enterprise.
* **What We Do:** 24/7 infrastructure uptime monitoring, core network configuration management, NetFlow telemetry tracking, and first-tier isolation of network-level security events before escalating to dedicated incident response.
* **Workforce & Management Model:** Hybrid model with a mix of fully remote staff, on-premise site operators, and hybrid workers; primary NOC operations run central shifts during business hours with a rotating tier-1 shift operating remotely during off-hours via dedicated VPN and SSH access.
* **Identity Management:** Centralized TACACS+/RADIUS server handling network CLI authentication across routers, switches, and firewalls; Active Directory manages standard Windows user endpoints and jump box credentials.
* **Infrastructure Layer:** Dual-vendor enterprise edge architecture (Cisco/Juniper routing cores) connected to Linux-based perimeter appliances, hybrid cloud connectors, and out-of-band management switches.
* **Vendor Access Protocols:** Strictly enforced out-of-band management plane requiring dual-factor VPN access to dedicated jump boxes before establishing internal SSH connections to core hardware.
* **The Environment Baseline:** Steady-state internal traffic consists primarily of standard HTTPS web traffic, local SMB file shares, encrypted administrative SSH from authorized jump boxes, and centralized internal DNS queries directed to primary domain controllers.


---

## Card Reveal Details
### Initial Access: Embedded RAT (Remote Access Trojan)
* Memory forensics on the marketing host reveals an orphaned, unnamed DLL injected into a legitimate system process that has no corresponding binary file on the local hard drive.
* Network connection logs show this running thread established an internal TCP session directly to port 22 of the perimeter router, acting as an in-memory relay agent.
* Event logs show the initial access relied on an embedded webshell framework left dormant inside an unmonitored legacy directory from a prior intrusion months ago.

### Persistence: SSH Authorized Keys Modification 
* Configuration analysis on the core edge router reveals an unknown RSA public key newly appended to the `/root/.ssh/authorized_keys` file.
* Audit trails indicate the key was added during an active interactive shell session originating directly from the infected internal marketing host.
* Device status checks show local password authentication was silently disabled for administrative users, forcing all future CLI access through this newly inserted SSH key pair.

### Pivot/Escalate: Dumping LSASS Via Task Manager
* Sysmon Event ID 11 logs on the marketing endpoint record a temporary `.dmp` file created inside the user's `AppData\Local\Temp` folder via a native `taskmgr.exe` execution thread.
* Process tracking logs show `taskmgr.exe` opened a direct memory handle with `PROCESS_VM_READ` rights targeting `lsass.exe` immediately before the dump file appeared.
* Security event logs show local domain admin credentials harvested from that memory space were reused minutes later to execute elevated network management commands across adjacent hosts.

### C2/Exfil: DNS as C2
* Core DNS server logs reveal thousands of outbound port 53 lookup requests sent to a series of high-entropy, dynamically generated subdomains under an unrated external domain.
* Packet inspection shows the payload data is heavily Base64-encoded and embedded directly within the TXT and A-record request headers rather than standard web traffic.
* Traffic rate metrics show these DNS queries are firing at irregular, low-frequency intervals specifically timed to bypass standard threshold-based network anomaly detectors.

---

## Heat Raisers
### Normal Incident
Turn 2: The Gateway Handshake
The Network Operations Center reports a critical anomaly on an internal distribution switch. An encrypted, high-privilege SSH connection was established targeting a core edge router, originating directly from a standard user workstation instead of an authorized administrative jump box.

Turn 6: The Dormant Process Dump
An automated security audit on a core Windows system highlights an isolated Sysmon file creation event. A process dump of the LSASS memory space (lsass.dmp) was briefly generated via Task Manager before being deleted, indicating local credential harvesting occurred on the host.

Turn 8: The Tunneling Anomaly
Security operations observes an influx of low-frequency, encoded DNS queries directed toward an unrated external domain. The query structures are formatted with high-entropy subdomains over port 53, confirming an active, covert command-and-control channel is operating.

Turn 10: Infrastructure Disruption (The Finale)
The threat actor executes their ultimate objective across the environment. Utilizing the persistent SSH access established on the network layer, the adversary deploys malicious firmware configurations across primary routing hardware, severing internal subnet communication, bricking core switches, and locking administrators out completely.


### Live Incident
* **Attacker Turn 0 (Turn 1): Hidden Event:** The threat actor wakes up a legacy embedded RAT left behind from a previous intrusion and drops a public SSH key into the core router's authorized_keys file to lock in persistent edge access.
* **Effect:** Players can discover the Initial Access and Persistence cards.

* **Attacker Turn 1 (Turn 2): Move:** The threat actor injects malicious code directly into legitimate, running host process memory spaces (`lsass.exe`/`svchost.exe`), bypassing static file scanners and complicating live memory analysis.
* **Effect:** -2 to next Endpoint Analysis roll.

* **Attacker Turn 2 (Turn 3): Hidden Event:** The adversary creates an lsass.dmp file via Task Manager to scrape administrative credentials, then initiates covert outbound C2 communications using encoded DNS queries.
* **Effect:** Players can discover the Pivot/Escalate and C2/Exfil cards.

* **Attacker Turn 3 (Turn 4): Move:** The adversary leverages native Windows administrative utilities (`LOLBins`) to harvest process dumps and bypass EDR behavioral heuristics.
* **Effect:** -2 to next Endpoint Security Protections Analysis roll.

* **Attacker Turn 4 (Turn 5): Event (Helpsdesk / NOC Alert):** The Network Operations Center reports that several core edge firewalls and WAN routers have unexpectedly entered administrative read-only mode. Local CLI logins are rejecting credential pushes, throwing the leadership team into chaos as basic network changes fail across all sites.
* **Effect:** -5 to Crisis Management rolls this turn.

* **Attacker Turn 5 (Turn 6): Move:** The threat actor alters local logging configurations on compromised jump boxes, redirecting Sysmon and audit streams to null loopback interfaces to blind centralized SIEM parsing.
* **Effect:** -2 to next SIEM Log Analysis roll.

* **Attacker Turn 6 (Turn 7): Event (Field / Perimeter Incident):** External incident response consultants attempt to access the environment via dedicated emergency management gateways, but their incoming IP blocks are systematically being dropped at the edge router. The attacker isactively actively rejecting external administrative connections before SSL/TLS negotiations complete.
* **Effect:** -5 to Call a Consultant rolls this turn.

* **Attacker Turn 7 (Turn 8): Move:** The adversary routes command-and-control communications using low-frequency, encoded DNS subdomains that mimic legitimate root name-server resolution patterns to evade standard network anomaly detection.
* **Effect:** -2 to next Network Threat Hunting roll.

* **Attacker Turn 8 (Turn 9): Event (Automated Network Audit):** An automated network compliance scanner flags an active configuration discrepancy across internal core switches. The threat actor has modified routing tables and established unauthorized logical bridges across internal VLANs, causing network isolation commands to fail as traffic automatically reroutes through rogue paths.
* **Effect:** -5 to Isolation rolls this turn.

* **Attacker Turn 9 (Turn 10): Event (Executive Operations Briefing):** The threat actor compromises the internal TACACS+/RADIUS authentication server configurations, attempting to intercept elevated network administrative credentials and blind authorization tracking right before launching their final disruption payload.
* **Bonus Event:** Internal network engineers combing through raw physical layer connections locate an exposed, unmonitored console port on a core distribution switch left open during the intrusion. Executive leadership secures immediate emergency funding and grants full authority to leverage this temporary high-visibility window.
* **Option 1:** +5 to one roll.
* **Option 2:** Players select three cards and can roll any combination of them with a -2 modifier applied to the rolls.
* **Option 3:** Incident Master chooses three cards (two incorrect, one correct); players must choose one and roll for it.

* **Attacker Turn 10 (End of Game - Failure): Event (Infrastructure Disruption):** Total infrastructure compromise occurs. The threat actor executes modified firmware commands across core networking hardware through their persistent SSH keys, bricking routing infrastructure, severing internal traffic streams, and locking administrators out of the environment entirely.

---

## Final Story
The intrusion began quietly, long before anyone in the Network Operations Center flagged a single anomaly. The threat actor woke up a legacy, embedded RAT that had been sitting completely dormant inside an unmonitored directory on a marketing workstation from a previous breach. The moment the backdoor initialized, they didn't waste time on the local host. Instead, they immediately pivoted to our core edge router, reaching into the file system to drop an unauthorized RSA public key straight into the /root/.ssh/authorized_keys file. That single move locked in a stealthy, persistent out-of-band foothold across our perimeter that completely bypassed standard identity and access controls.

To secure their position on the internal network without triggering local Endpoint Detection and Response alerts, the adversary turned to native Windows administrative tools. They injected malicious payload code directly into the active memory spaces of legitimate system processes like lsass.exe and svchost.exe. Operating entirely in-memory, they used Task Manager to spawn a process dump file (lsass.dmp), harvested plaintext domain administrative credentials from memory, and immediately wiped the dump file to avoid static file scanners. With those high-privilege credentials secured, the attackers established a low-and-slow C2 channel over DNS, trickling encoded commands and exfiltration data over port 53 via high-entropy subdomains designed to mimic legitimate root name-server traffic.

Note: Early Ending (If the players successfully contain the incident around Turn 5 or 6, stop reading here and jump to Ending 3: Midgame Victory)

As the intrusion escalated into a full-scale crisis, the adversary launched a coordinated effort to blind our operational response. They modified administrative parameters on our core edge firewalls and WAN gateways, forcing the hardware into read-only mode and rejecting local credential pushes. Leadership was thrown into disarray as basic network isolation changes failed across all sites. On the compromised jump boxes, the threat actor altered local logging parameters, redirecting Sysmon and audit streams directly to null loopback interfaces to cut off telemetry to our central SIEM. When our leadership team attempted to bring in external incident response consultants to salvage the perimeter, the attacker actively dropped incoming connection blocks at the edge router before SSL/TLS negotiations could even complete.

In the late stage of the campaign, the threat actor engineered total chaos across our physical and logical switching layers. By modifying internal routing tables and setting up unauthorized logical bridges across isolated subnets, they rendered our standard network isolation playbooks useless; any attempt by our NOC to wall off compromised zones resulted in traffic silently rerouting through their rogue paths. In a final push to solidify absolute control, the adversary compromised our central TACACS+ and RADIUS authentication servers, attempting to intercept active administrative tokens and blind our authorization tracking right before deploying their final disruption payload.

Just as our team was facing complete system lockout, our internal network engineers achieved a crucial physical-layer breakthrough. Combing through raw distribution hardware connections, they located an exposed, unmonitored console port on a core switch that had been left open during the intrusion, granting our incident response team a brief, high-visibility window of raw telemetry to turn the tide.


### Ending 1: Victory (The Successful Eviction)
Armed with that high-visibility console access, our team caught the adversary mid-stride. We traced the rogue DNS queries back to the compromised marketing host, isolated the system, and killed the embedded RAT process before it could re-establish its foothold. With the endpoint contained, we purged the unauthorized SSH public key from the core router, revoked the compromised TACACS+ credentials, and restored our edge configurations from known-good backups. The threat actor was completely severed from our environment, leaving our network infrastructure intact and fully operational.


### Ending 2: Failure (Total Domain Compromise)
Before our team could act on the raw console telemetry, the adversary made their final move. Using the persistent SSH keys they dropped early on, the threat actor pushed malicious firmware configurations across our entire core routing plane. Within seconds, primary switches went dark, routing tables were wiped, and administrative access to the CLI was permanently locked out across every site. The attack bricked our core network hardware, severed internal subnet communications, and left us completely blind, delivering the total infrastructure collapse Sandworm aimed for from the start.


### Ending 3: Midgame Victory (Early Containment)
By moving quickly and zeroing in on the network edge early, our team caught the threat actor before they could lock down the environment. We caught the suspicious SSH handshake to the core router, identified the dormant RAT on the marketing box, and wiped the rogue public key from the authorized_keys file before the attacker could fully pivot into our logging infrastructure. With the DNS tunneling channel blocked and the embedded backdoor scrubbed from memory, we cut off their foothold before any major operational disruption hit our core switches, keeping the entire network safe and online.

Thanks so much for coming out, engaging with the scenario, and making this session awesome. I hope you picked up a few new tricks and had a great time working through the chaos together!
