# Security Investigations Portfolio

This repository serves as a practical portfolio of full-cycle security investigations, threat hunting exercises, and log analysis case studies. The objective of these writeups is to demonstrate hands-on proficiency in identifying, analyzing, and documenting cyber threats across various attack vectors, mirroring the day-to-day operations of a Security Operations Center (SOC) analyst.

By reconstructing attack timelines from raw telemetry and packet captures, these case studies highlight an analytical methodology focused on root cause analysis, Indicator of Compromise (IOC) extraction, and actionable remediation.

## 📊 Case Studies

### 1. [WebStrike: Web Application Compromise & Exfiltration](./case-studies/01-webstrike/README.md)
* **Platform:** CyberDefenders
* **Tools Used:** Wireshark
* **Highlights:** Investigated a web server breach by analyzing HTTP/TCP streams. Identified an initial access bypass using a double-extension file upload (`.jpg.php`), traced the execution of a reverse shell over port 8080, and documented the subsequent exfiltration of system files (`/etc/passwd`) via outbound `curl` POST requests. 

### 2. [Oski Lab: Stealc Malware Alert](./case-studies/02-Oski/README.md)
* **Platform:** CyberDefenders
* **Tools Used:** VirusTotal, Any.run
* **Highlights:** Investigated a Stealc/Oski malware infection originating from a malicious PPT attachment. Utilized threat intelligence platforms to trace C2 communication, extract the RC4 decryption key, and map credential harvesting behavior to MITRE ATT&CK T1555. Analyzed evasion tactics, including timed self-deletion commands and DLL removal in the ProgramData directory.

---

**Last Updated:** 24-Apr-26

---

## 📬 Contact & Connect
I am currently pursuing a degree in Cybersecurity and actively expanding my defensive security skill set through continuous lab work and CTF engagements.

* [LinkedIn](https://www.linkedin.com/in/syed-sheharyar-ahmed-32479b376/)
* **Email:** sheharyarahmedsyed@gmail.com 
