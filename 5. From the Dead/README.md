# From the Dead
## Scenario
```
[TIMESTAMP: 08:14:22 UTC]
[SOURCE IP: 10.20.4.112 - Host: dev-workstation-09.internal.bybit2.net]
[TARGET URI: https://staging-app.bybit2.internal/assets/vendor/bundle.js]

ALERT DETAILS:
The Web Application Firewall detected an unexpected outbound HTTP request originated 
by an internal developer staging application. 

Details:
1. Static asset bundle (bundle.js) rendered on dev-workstation-09 failed 
   Subresource Integrity (SRI) check.
2. The modified script attempted to fetch an external dependency from an unverified 
   domain: [ cdn.open-js-registry[.]net/analytics.js ]
3. No active deployment ticket was found matching this modification.

STATUS: Flagged for analyst review. Application execution permitted (Monitor Mode).
```

---

## Relevant Details
* **Who We Are:** Bybit2 is a major global centralized digital asset exchange handling high-volume spot trading, derivatives, and institutional treasury operations.
* **What We Do:** We manage liquidity pools, execute real-time order matching, and facilitate large-scale asset transfers between hot trading wallets and multi-signature cold storage vaults.
* **Workforce & Management Model:** Core platform engineering and treasury teams operate out of centralized regional hubs, but we rely heavily on trusted, third-party open-source libraries, external Web3 developer tooling, and hosted frontend asset repositories to manage our web interfaces.
* **Identity Management:** We utilize a hybrid identity structure combining cloud-based IAM for developer pipeline environments with hardware-backed Multi-Factor Authentication (MFA) and API tokens for internal administrative portals.
* **Infrastructure Layer:** The operational ecosystem relies on a high-throughput microservices architecture hosted across public cloud infrastructure (AWS) alongside specialized smart contract interaction endpoints for on-chain asset movements.
* **Treasury & Signing Protocols:** Transfers out of corporate cold storage require a multi-signature (multisig) approval workflow. Authorized signers use dedicated management web portals to review and cryptographically sign transaction payloads before broadcasting them.
* **The Environment Baseline:** Our web applications enforce strict change control and build verification protocols. Any unexpected external resource fetch, failed Subresource Integrity check, or unapproved modification to static asset bundles instantly triggers an operational security alert.

---

## Card Reveal Details
### Supply Chain Compromise (Initial Access)
* **Poisoned Open-Source Dependency:** Investigators identified a compromised third-party JavaScript package hosted on a public repository that was pulled directly into Bybit2's staging build environment.
* **Tampered Subresource Integrity (SRI):** Static asset checks flagged that the fingerprint of a core utility library bundle failed its expected validation hash immediately after a routine developer update.
* **Obfuscated Payload Delivery:** The malicious script was hidden deep inside a benign-looking analytics wrapper, executing only when specific internal environment parameters matched Bybit2's staging domain.
* **Local Developer Machine Execution:** The initial code execution was isolated to a single developer workstation, running quietly with standard user permissions during a normal software build cycle.

### Access Token Manipulation (Pivot / Escalate)
* **Memory & Environment Scraping:** Deeper inspection of the developer workstation revealed a lightweight script designed to harvest active session tokens, local environment variables, and cached authentication keys.
* **CI/CD Pipeline Takeover:** The threat actor used the stolen high-privilege tokens to authenticate directly into Bybit2's deployment pipeline without triggering traditional perimeter multi-factor authentication.
* **Bypassing Code Review Boundaries:** By using valid access tokens within the trusted build pipeline, the actor pushed unauthorized code modifications directly into application asset repositories without needing a secondary peer sign-off.
* **Persistent Token Renewal:** Network logs showed programmatic requests periodically refreshing the stolen session tokens from a remote host to maintain access across standard change windows.

### Malicious Browser Plugin (Persistence)
* **Trusted Process Execution:** Forensics on a key administrative machine uncovered a rogue browser extension running entirely inside the active memory space of a legitimate, trusted web browser process.
* **Presentation Layer Tampering:** The extension was designed to silently inspect and intercept active Document Object Model (DOM) data strictly when administrative users accessed Bybit2's internal web interfaces.
* **Silent Session Hijacking:** Instead of dropping persistent files on the disk, the plugin monitored active sessions to capture transaction approval requests in real-time while bypassing local endpoint antivirus software.
* **Targeted Web Interface Masking:** The script contained logic to alter what was rendered on the screen during administrative workflows while keeping all standard system features operating without visual lag.

### Domain Fronting (C2 / Exfil)
* **CDN Header Manipulation:** Network traffic analysis revealed outbound HTTPS calls utilizing modified HTTP Host headers to route traffic through a legitimate, high-reputation Content Delivery Network (CDN).
* **Encrypted Traffic Blending:** Command and control communications were entirely wrapped in standard TLS encryption, making the malicious traffic look like routine external web application asset requests.
* **Evasion of Domain Reputation Filters:** By leveraging the trusted domain of a major public cloud CDN, the outbound payload requests completely bypassed standard firewall and web proxy blocking lists.
* **Covert Instruction Staging:** The encrypted channel was used to send real-time configuration instructions down to the application layer, staging the eventual destination address swap for high-value operations.

---

## Heat Raisers
### Normal Incident 
- Turn 3 (Attacker Turn 4): Whispering starts circulating in the internal #dev-ops and #engineering Slack channels. A few developers report that their IDEs are running unusually slow and auto-syncing packages to staging servers without active PR approvals.

- Turn 5 (Attacker Turn 6): The SOC team receives an urgent external threat intelligence flash from an industry ISAC group. A major crypto platform just suffered a silent API token harvest stemming from malicious open-source packages.

- Turn 7 (Attacker Turn 8): The VP of Infrastructure pops into the war room asking for a status report. They mention that leadership is preparing for a scheduled multi-million dollar asset rebalancing from cold storage to hot wallets later today, asking if the systems are totally clear to proceed.

- Turn 9 (Attacker Turn 10): Network traffic suddenly drops to near-normal levels, and the previous noisy alerts stop firing. However, automated pre-transaction logs show an active administrative session logging into the main treasury interface.


### Live Incident
* Turn 0 (Attacker Turn 1): Event: Compromises a popular open-source JavaScript dependency used by Bybit2 developers, while simultaneously establishing a quiet C2 beacon over outbound HTTPS.
  - Effect: Defenders can now discover Initial Access and C2/Exfil

- Turn 1 (Attacker Turn 1): Move: The threat actor uses stolen administrative credentials to silently modify sensor configurations and unhook agent monitoring functions on the initial build host.
  - Effect: Defenders take a -2 penalty on Endpoint Security Protection Analysis rolls this turn.

* Turn 2 (Attacker Turn 3): Event: Uses harvested environment variables from a developer workstation to pivot into the CI/CD pipeline, quietly staging a malicious browser extension to maintain long-term persistence across core dev environments.
  - Effect: Defenders can now discover Pivot/Escalate and Persistence

- Turn 3 (Attacker Turn 3): Move: The threat actor executes payload components entirely in system memory and timestomps file creation dates across compromised developer workstations.
  - Effect: Defenders take a -2 penalty on Endpoint Analysis rolls this turn.

* Turn 4 (Attacker Turn 5): Event: The threat actor executes a script that floods network loggers and administrative consoles with false routing changes, forcing IT to lock down internal routing tables. Defenders cannot use Isolation cards this turn while the network team stabilizes the infrastructure.
  - Effect: Defenders can't use the Isolation card this turn

- Turn 5 (Attacker Turn 5): Move: The threat actor leverages domain fronting over trusted public CDN networks, masking command and control traffic inside standard TLS encrypted web traffic.
  - Effect: Defenders take a -2 penalty on Firewall Log Analysis rolls this turn.

* Turn 6 (Attacker Turn 7): Event: The attacker deploys a targeted wiper or ransomware lure on executive laptops to create a massive distraction. Defenders cannot use Crisis Management cards this turn as leadership is completely tied up dealing with the executive-level outage.
  - Effect: Defenders can't use the Crisis Management card this turn

- Turn 7 (Attacker Turn 7): Move: The threat actor generates randomized, low-volume background traffic and jittered beacon intervals across multiple internal developer subnets.
  - Effect: Defenders take a -2 penalty on Network Threat Hunting rolls this turn.

* Turn 8 (Attacker Turn 9): Event: The threat actor severly tampers with external API endpoints and third-party portal authentication tokens, dropping external VPN connections. Defenders cannot use Call a Consultant cards this turn because external responders cannot securely access the environment.
  - Effect: Defenders can't use the Call a Consultant card this turn

- Turn 9 (Attacker Turn 9): Move: The threat actor triggers automated event floods from staging environment endpoints, causing central log collectors to drop incoming event frames.
  - Effect: Defenders take a -2 penalty on SIEM Log Analysis rolls this turn.

* Turn 10 (Attacker Turn 11): Event: The final payload activates in the executive portal interface. The destination wallet address is silently swapped during an automated transfer request, initiating the drain of $1.5 billion in Ethereum to the Lazarus pool.
  - Management Bonus Event: Emergency Treasury Authorization. Realizing the firm is minutes away from total insolvency, executive management opens the floodgates. The defenders get an emergency budget boost, giving them unlimited funds or 2 extra roll bonuses on their final turn to identify and block the rogue transaction before final block confirmation.

---

## Final Story
### Full Story
The incident begins when an attacker quietly poisons a popular open-source JavaScript dependency used across Bybit2 developer environments. At the same time, a hidden command and control beacon establishes outbound traffic over standard HTTPS. Shortly after, the threat actor uses harvested administrative credentials to quietly alter sensor configurations and unhook agent monitoring functions on the initial build host. This makes endpoint security protection analysis much harder for defenders early on.

With initial access set, the threat actor extracts environment variables from a developer workstation. They use these stolen tokens to pivot directly into Bybit2's CI/CD pipeline, staging a malicious browser extension to maintain persistent access across development environments. To cover their tracks, the attacker executes payload components strictly inside system memory and timestomps file creation dates across compromised workstations, creating heavy resistance against endpoint analysis.

The attack escalates as a malicious script floods network loggers and admin consoles with fake routing updates. IT teams have to lock down internal routing tables to stabilize infrastructure, which completely blocks defenders from playing Isolation cards. While defenders scramble, the threat actor routes command traffic through public CDN networks using domain fronting, disguising traffic inside standard TLS sessions and complicating firewall log analysis.

To draw attention away from the core target, the attacker drops a targeted wiper payload onto executive laptops. Leadership gets completely tied up handling the sudden executive outage, locking out Crisis Management cards. Meanwhile, the threat actor injects randomized, low-volume background traffic and jittered beacon intervals across developer subnets, throwing a major curveball at network threat hunting efforts.

Approaching the final objective, the threat actor tampers with external API endpoints and third-party portal tokens, dropping external VPN connections. External incident response teams can't connect, effectively disabling Call a Consultant cards. The attacker then triggers automated log floods from staging endpoints to overwhelm central log collectors, causing SIEM systems to drop incoming event frames and hindering SIEM log analysis.

### Failed Ending
The threat actor triggers their primary objective inside the executive web portal. When executives initiate a routine multisig transfer, the malicious browser extension renders legitimate transaction details on screen while altering the underlying contract payload. A massive transfer of 1.5 billion dollars in Ethereum gets signed via blind signature and broadcast to the blockchain. Even with the Emergency Treasury Authorization bonus granting last-minute defensive boosts, defenders fail to isolate the modified payload before final block confirmation. The funds drain into the attacker wallet pool, leaving Bybit2 facing total insolvency.


### Early Success
Defenders successfully catch the subresource integrity alert on the compromised JavaScript dependency and trace the stolen access tokens in the CI/CD pipeline. By identifying the malicious browser plugin early on, the incident response team revokes all active session tokens and isolates the compromised build host before the domain fronting C2 channel fully establishes. The team purges the poisoned dependency from the staging repository, killing the attacker's persistent access and stopping the threat before any executive systems or treasury portals are impacted.


### Midnight Hour
Despite dealing with locked routing tables, executive disruptions, and dropped SIEM logs, defenders hold out until Turn 10. When executive management issues the Emergency Treasury Authorization, the defense team gains critical extra roll bonuses at the eleventh hour. Responders correlate the domain fronting C2 indicators with the modified DOM elements in the executive web interface right as the multisig transfer is submitted. The team halts the transaction at the network node level, revokes the compromised signing portal tokens, and prevents the 1.5 billion dollar payload from reaching block confirmation on the blockchain.
