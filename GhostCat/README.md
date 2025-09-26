# 🔐 TomGhost Machine Penetration Testing Report

*"Remember, remember, the fifth of November..."* 
Welcome to my detailed penetration testing report for the TomGhost machine from TryHackMe. 
This repository contains a comprehensive walkthrough of my journey from initial reconnaissance
to full root compromise using the infamous GhostCat vulnerability.

## 📖 Overview

This write-up demonstrates a real-world attack chain exploiting:
- Apache Tomcat GhostCat vulnerability (CVE-2020-1938)
- PGP credential decryption weaknesses
- Privilege escalation via ZIP sudo misconfiguration

**Perfect for intermediate learners exploring web services exploitation and Linux privilege escalation techniques!**

---

## 🎯 Quick Facts
- **Difficulty**: Intermediate 🔥
- **Platform**: Linux 🐧
- **Key Skills**: Web Enumeration, Vulnerability Research, Privilege Escalation
- **Tools Used**: Nmap, GhostCat Exploit, GPG, LinPEAS

## 🚀 Attack Flow
1. **Reconnaissance** → Port scanning & service discovery
2. **Exploitation** → GhostCat file read vulnerability
3. **Initial Access** → PGP credential decryption
4. **Privilege Escalation** → ZIP sudo command injection

## 💡 Key Takeaways
- Always patch known vulnerabilities like CVE-2020-1938
- Secure sensitive credentials and encryption keys
- Audit sudo permissions regularly
- Understand how seemingly harmless binaries can be weaponized

---

*"Ideas are bulletproof."* 🎭
