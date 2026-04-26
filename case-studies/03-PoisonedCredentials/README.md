# LLMNR/NBT-NS Poisoning & Credential Theft

## 📖 Executive Summary
* **Platform/Dataset:** [CyberDefenders](https://cyberdefenders.org/blueteam-ctf-challenges/poisonedcredentials/) - Custom PCAP (`PoisonedCredentials.pcap`) 
* **Category:** Credential Harvesting / LLMNR & NBT-NS Poisoning
* **Status:** ✅ Completed

### Situation
The SOC identified suspicious network traffic indicating potential name resolution spoofing within the environment. Analysis of the packet capture (`PoisonedCredentials.pcap`) revealed an attacker taking advantage of benign network traffic and mistyped queries to conduct an active Active Directory attack.

### Task
1. Analyze the packet capture to identify the mistyped network queries
2. Locate the rogue entity's IP address
3. Determine the scope of affected machines
4. Identify the compromised user account and targeted host machine.

---

## 🔍 Investigation & Action (Methodology)

1. **Initial Triage & Query Analysis:** Filtered the PCAP for the initial victim's IP (`192.168.232.162`) to isolate relevant traffic. Discovered that the user attempted to navigate to a file share but mistyped the query as `FILESHAARE` instead of `Fileshare`.
   <img width="1393" height="703" alt="image" src="https://github.com/user-attachments/assets/fec24da4-32dd-4d58-83d3-350081dabddb" />
2. **Rogue Machine Identification:** Analyzed the responses to the mistyped broadcast queries. Identified the IP address `192.168.232.215` responding maliciously to the `fileshaare` broadcast, as well as a separate `prinetr` query. This confirmed `192.168.232.215` as the rogue entity.
   <img width="1221" height="362" alt="image" src="https://github.com/user-attachments/assets/26232f72-57fb-494b-870d-34b3f1d8c8dc" />

3. **Scope of Impact Assessment:** Expanded the investigation to find other machines receiving poisoned responses. Identified a second victim machine at `192.168.232.176` that received a poisoned response from the rogue machine after querying for `prinetr`.
    <img width="1180" height="360" alt="image" src="https://github.com/user-attachments/assets/2602e47b-c10f-4649-9f1d-cb8a2789a4eb" />

4. **Credential Compromise Verification:** Queried all packets related to the attacker's IP (`192.168.232.215`) to assess post-exploitation activity. Located an SMB2 session setup request confirming the attacker captured credentials and successfully authenticated as the user `janesmith`.
    <img width="1338" height="717" alt="image" src="https://github.com/user-attachments/assets/ef6563d9-e90a-4070-adee-dac771c8f95f" />

5. **Target Host Identification:** Inspected the SMB2 Session Setup Response headers to map the attacker's lateral movement. Confirmed the compromised account belongs to the `cybercactus.local` domain and the attacker accessed the hostname `WORKSTATION`.
    <img width="1524" height="855" alt="image" src="https://github.com/user-attachments/assets/52798904-cfef-4dae-bb20-39c4088133d6" />
 

---

## ⏱️ Incident Timeline
*A chronological reconstruction of the attack based on the packet capture.*

| Event | Source / Evidence |
| :--- | :--- |
| Victim 1 (`192.168.232.162`) broadcasts a mistyped name query for `FILESHAARE`. | NBNS Name Query |
| Rogue machine (`192.168.232.215`) sends a poisoned response claiming to be `fileshaare`. | LLMNR Standard Query Response |
| Rogue machine sends a poisoned response claiming to be `prinetr` to Victim 2 (`192.168.232.176`). | LLMNR Standard Query Response |
| Attacker initiates SMB Session Setup using compromised credentials for `cybercactus.local\janesmith`. | SMB2 Session Setup Request |
| Successful SMB authentication to `WORKSTATION`. | SMB2 Session Setup Response |


---

## 🛑 Root Cause Analysis
The root cause of this incident was the enabled state of legacy broadcast resolution protocols (LLMNR and NBT-NS) on the local network. This configuration allowed the attacker to listen for mistyped network resource requests (`FILESHAARE` and `prinetr`) and rapidly respond with their own IP address before the legitimate DNS server could respond. Consequently, the victims' machines automatically attempted to authenticate with the attacker's rogue server, exposing the NTLM hashes for the `janesmith` account, which were then used to establish an unauthorized SMB session on `WORKSTATION`.

---

## 🛡️ Result & Remediation

**Immediate Containment Actions:**
* Isolate the rogue IP (`192.168.232.215`) from the network.
* Force a password reset for the compromised domain account (`janesmith`).
* Isolate the affected endpoints (`192.168.232.162`, `192.168.232.176`, and `WORKSTATION`) to investigate potential lateral movement or dropped payloads.

**Long-Term Recommendations:**
* **Disable LLMNR and NBT-NS:** Implement Group Policy Objects (GPOs) to disable LLMNR (Turn off multicast name resolution) and disable NetBIOS over TCP/IP across all workstations and servers.
* **Require SMB Signing:** Enforce SMB signing across the domain to prevent NTLM relay attacks from successfully authenticating to other machines.
* **Implement Network Segmentation:** Ensure sensitive workstations and servers are segmented from general user vlans to restrict unauthorized local broadcast traffic.
