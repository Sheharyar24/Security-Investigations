# Oski Lab - Stealc Malware Alert

## 📖 Executive Summary
* **Platform/Dataset:** CyberDefenders 
* **Category:** Threat Intel
* **Status:** ✅ Completed
* **Objective:** Analyze the sandbox report and provided malware hash using threat intelligence platforms (VirusTotal, Any.Run) to identify the Stealc malware's behavior, extract its configuration details, map the observed tactics to MITRE ATT&CK, and provide actionable remediation steps.

### Situation
An accountant received an email titled **"Urgent New Order"** from a client late in the afternoon. When they attempted to access the attached invoice (a PPT file), it contained false order information. Subsequently, the SIEM generated an alert regarding the download of a potentially malicious file triggered by the PPT attachment.

### Task
1. Determining the creation time of the malware 
2. Identifying command and control (C2) connection
3. Identifying the initial actions of the malware post-infection
4. Identify the main MITRE attack technique
5. Understanding the malware's behavior post-data exfiltration

---

## 🔍 Investigation & Action (Methodology)

1.  **Initial Triage:** Extracted the initial 140-Oski.zip archive to retrieve hash.txt. Identified the primary MD5 hash for the investigation as `12c1842c3ccafe7408c23ebf292ee3d9`.
    <img width="724" height="257" alt="image" src="https://github.com/user-attachments/assets/ff967189-e99c-432c-b190-3c96c6dda04c" />
 
2. **Threat Intelligence Correlation** Queried VirusTotal using the extracted hash. Identified the malware's creation time as `2022-09-28 17:40:46 UTC`.
    <img width="529" height="144" alt="image" src="https://github.com/user-attachments/assets/14ddfe82-0918-48d0-96aa-802847eec5c2" />

3.  **Network Activity Analysis:** Examined the 'Relations' tab in VirusTotal to trace outbound connections. Discovered the malware communicates with a C2 server at `http://171.22.28.221/5c06c05b7b34e8e6.php`.
    <img width="757" height="130" alt="image" src="https://github.com/user-attachments/assets/8483365a-ff57-4718-81a7-cf8dbc00d97e" />

4.  **Sandbox Execution Analysis (Any.run):** Reviewed the Any.run sandbox report and noted that the first library requested by the malware post-infection was `sqlite3.dll`.
    <img width="1338" height="286" alt="image" src="https://github.com/user-attachments/assets/191a9d6a-09e6-41a8-bf1b-e231a1b81270" />
   
5.  **Configuration Extraction:** Analyzed the malware configuration block within the sandbox report and extracted the RC4 key used for decryption: `5329514621441247975720749009.`
    <img width="1127" height="277" alt="image" src="https://github.com/user-attachments/assets/98168892-fbae-4b36-9164-c384526a600c" />

6.  **Behavioral Mapping:** Identified that the Stealc/Oski malware exfiltrates files and passwords, mapping this primary behavior to MITRE ATT&CK technique `T1555` (Credentials from Web Browsers/Password Stores).
    <img width="516" height="479" alt="image" src="https://github.com/user-attachments/assets/27661e5b-18a0-4938-9926-3b8938184568" />

7. **Evasion Tactics Identification:** Analyzed child processes and found that the malware spawns a CMD process that waits 5 seconds before self-deleting its executable (VPN.exe). It simultaneously runs a command to delete all DLL files located inside the ProgramData directory.
    <img width="602" height="288" alt="image" src="https://github.com/user-attachments/assets/610a4b8f-af7e-4f2b-91b9-7135032fb08b" />


---

## 🛑 Root Cause Analysis & Remediation
The incident originated from a social engineering attack where an employee interacted with a malicious PPT attachment delivered via a phishing email. The root cause was the lack of execution restrictions on payloads originating from Office applications, which allowed the PPT file to drop and execute the Stealc malware. Once executed, the malware harvested credentials from web browsers and password stores (`T1555`), established external C2 communication, and executed a 5-second self-deletion command to remove its binary (VPN.exe) and associated DLLs in ProgramData to evade forensic detection.

**Remediation Recommendations:**
* Implement enhanced email filtering rules to flag or quarantine external emails containing `Urgent` keywords paired with standard office attachments.
* Deploy a custom SIEM alert or EDR rule to detect `cmd.exe` spawning with timeout commands immediately followed by file deletion (`del /f /q`) in **AppData or ProgramData** directories.
* Enforce Multi-Factor Authentication (MFA) across all external access and internal movement to mitigate the impact of stolen credentials.
