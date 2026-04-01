# Security Investigations Portfolio

This repository serves as a practical portfolio of full-cycle security investigations, threat hunting exercises, and log analysis case studies. The objective of these writeups is to demonstrate hands-on proficiency in identifying, analyzing, and documenting cyber threats across various attack vectors, mirroring the day-to-day operations of a Security Operations Center (SOC) analyst.

By reconstructing attack timelines from raw telemetry and packet captures, these case studies highlight an analytical methodology focused on root cause analysis, Indicator of Compromise (IOC) extraction, and actionable remediation.

## 📊 Case Studies

### 1. [WebStrike: Web Application Compromise & Exfiltration](./case-studies/01-webstrike/README.md)
* **Platform:** CyberDefenders
* **Tools Used:** Wireshark
* **Highlights:** Investigated a web server breach by analyzing HTTP/TCP streams. Identified an initial access bypass using a double-extension file upload (`.jpg.php`), traced the execution of a reverse shell over port 8080, and documented the subsequent exfiltration of system files (`/etc/passwd`) via outbound `curl` POST requests. 

---

**Last Updated:** 1-Apr-26

---

## 📬 Contact & Connect
I am currently pursuing a degree in Cybersecurity and actively expanding my defensive security skill set through continuous lab work and CTF engagements.

* [LinkedIn](https://www.linkedin.com/in/syed-sheharyar-ahmed-32479b376/)
* **Email:** sheharyarahmedsyed@gmail.com 
