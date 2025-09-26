# Penetration Testing Report - TomGhost Machine 🔥

![TomGhost](https://img.shields.io/badge/Level-Intermediate-🟢) ![Platform](https://img.shields.io/badge/Platform-Linux-🔴) ![Category](https://img.shields.io/badge/Category-CTF-🔵) ![Status](https://img.shields.io/badge/Status-COMPROMISED-🟣)

## 📋 Table of Contents
- [🎯 Executive Summary](#executive-summary)
- [🔍 Methodology](#methodology)
- [🕵️ Reconnaissance](#reconnaissance)
- [💥 Exploitation](#exploitation)
- [🔮 Post-Exploitation](#post-exploitation)
- [⚡ Privilege Escalation](#privilege-escalation)
- [📊 Conclusion](#conclusion)
- [🎓 Lessons Learned](#lessons-learned)

## 🎯 Executive Summary

**🎯 Target Machine**: TomGhost  
**🌐 IP Address**: `10.10.19.154` (from SSH connection)  
**💻 Operating System**: Ubuntu 16.04.6 LTS (Xenial)  
**⚙️ Kernel Version**: 4.4.0-174-generic  
**📊 Difficulty Level**: Intermediate  
**🎯 Objective**: Obtain user and root flags 🚩

**⚠️ Overall Risk Level**: **HIGH** 🔴  
**🛡️ Compromise Status**: **FULLY COMPROMISED** 🏴 (Root access achieved)

**🚩 Flags Captured:**
- 🧑‍💻 User Flag: `THM{GhostCat_1s_so_cr4sy}`
- 👑 Root Flag: `THM{Z1P_1S_FAKE}`

## 🔍 Methodology

This penetration test followed a structured approach 🔄:

1. **🔍 Information Gathering** - Network scanning and service enumeration
2. **🔓 Vulnerability Analysis** - Identifying weaknesses in exposed services
3. **💥 Exploitation** - Gaining initial foothold through Apache Tomcat vulnerability
4. **🔮 Post-Exploitation** - Lateral movement and data collection
5. **⚡ Privilege Escalation** - Obtaining root privileges through sudo misconfiguration

## 🕵️ Reconnaissance

### 🌐 Network Scanning

Initial port scanning revealed several open services 🚪:
- **🔒 Port 22/SSH**: OpenSSH service
- **🌐 Port 53/DNS**: DNSmasq service
- **🐱 Port 8080/HTTP**: Apache Tomcat service
- **⚡ Port 8009/Tomcat AJP**: Apache JServ Protocol listener

### 🔧 Service Enumeration

**Apache Tomcat 8.5.5** 🐱 was identified running on port 8080. This version is vulnerable to the **GhostCat vulnerability (CVE-2020-1938)**, which allows arbitrary file reading through the AJP protocol.

### 🔓 Vulnerability Assessment

The **GhostCat vulnerability (CVE-2020-1938)** was identified as the primary attack vector 🎯. This vulnerability exists in the AJP connector of Apache Tomcat and allows attackers to read web application files from the server, including configuration files and source code.

## 💥 Exploitation

### 👻 Initial Foothold via GhostCat

Using the GhostCat exploit, critical files were read from the Tomcat server 📄:

1. **🛠️ Exploitation Tool**: GhostCat exploit script (CVE-2020-1938)
2. **🎯 Target Files**: 
   - `/home/skyfuck/tryhackme.asc` - PGP private key file 🔑
   - `/home/skyfuck/credential.pgp` - Encrypted credentials file 🔒

**📂 Files Retrieved:**
- PGP private key for user "tryhackme" 🔑
- Encrypted credential file 🔒

### 🔓 Access as User Skyfuck

The retrieved PGP key and encrypted file were used to gain access to the system 💻:

```bash
# 🎯 Import the PGP key
gpg --import tryhackme.asc

# 📁 Set up temporary GPG home directory
mkdir -p /tmp/gpghome
export GNUPGHOME=/tmp/gpghome

# 🔑 Import the private key
gpg --import tryhackme_privkey.asc

# 🔓 Decrypt the credential file
gpg --output /tmp/credential.txt --decrypt credential.pgp
```

**🔓 Decrypted Credentials:**
- 👤 Username: `merlin`
- 🔑 Password: `asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j`

## 🔮 Post-Exploitation

### 🔍 System Enumeration

After gaining initial access as user `skyfuck`, comprehensive system enumeration was performed 🔎:

**🛠️ Tools Used:**
- `linpeas.sh` - Linux privilege escalation auditing script 📊
- `pspy64` - Process monitoring tool 👀
- `linux-exploit-suggester.sh` - Kernel vulnerability identification 🔍

**🔑 Key Findings:**
- Ubuntu 16.04.6 LTS with kernel 4.4.0-174-generic
- Multiple privilege escalation vectors identified ⚡
- ⏰ Cron job running as root: `* * * * * root cd /root/ufw && bash ufw.sh`
- 🐱 Tomcat service running as user `tomcat`

### 🚩 User Flag Capture

```bash
# 📂 Access merlin's home directory
cat /home/merlin/user.txt
THM{GhostCat_1s_so_cr4sy}
```

## ⚡ Privilege Escalation

### 🔄 Lateral Movement to Merlin

Using the decrypted credentials, access was obtained to user `merlin` 🔓:

```bash
su merlin
Password: asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j
```

### 🔍 Sudo Privilege Analysis

Checking sudo permissions for user `merlin` revealed a critical misconfiguration ⚠️:

```bash
sudo -l
User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
```

### 👑 Root Access via ZIP Command Injection

The ZIP binary allowed command execution as root through a technique involving temporary files and command injection 💉:

```bash
# 📁 Create a temporary file pattern
TF=$(mktemp -u)

# 💥 Execute zip with command injection
sudo zip $TF /etc/hosts -T -TT 'sh #'
```

**🔧 Exploitation Details:**
- The `-T` option allows testing the archive integrity
- The `-TT` option specifies a command to execute for testing
- By injecting `sh #` as the test command, a root shell is spawned 🐚

### 👑 Root Flag Capture

```bash
# 📂 Access root directory
cd /root
ls
cat root.txt
THM{Z1P_1S_FAKE}
```

## 📊 Conclusion

The TomGhost machine was successfully compromised through a methodical penetration testing approach 🎯. The attack path demonstrates a classic web application to root compromise scenario:

### 🔗 Attack Chain Summary:
1. **🔍 Reconnaissance** → Tomcat service identification and vulnerability assessment
2. **💥 Exploitation** → GhostCat vulnerability exploitation to read sensitive files
3. **🔓 Initial Access** → PGP decryption to obtain user credentials
4. **🔄 Lateral Movement** → Access to user merlin via decrypted password
5. **⚡ Privilege Escalation** → Sudo misconfiguration exploitation to gain root access

### 🚨 Security Vulnerabilities Exploited:
1. **CVE-2020-1938 (GhostCat)** 👻 - Arbitrary file read in Apache Tomcat
2. **🔓 Weak Credential Storage** - Passwords stored in encrypted files without proper protection
3. **⚡ Sudo Misconfiguration** - ZIP binary allowed to run as root without password

## 🎓 Lessons Learned

### 🚨 Critical Security Issues Identified

1. **📅 Outdated Software Components**
   - Apache Tomcat 8.5.5 with known vulnerabilities
   - Unpatched GhostCat vulnerability (CVE-2020-1938)

2. **🔓 Poor Credential Management**
   - Sensitive credentials stored in encrypted files accessible via web vulnerability
   - Weak password encryption mechanisms

3. **⚡ Misconfigured Privileges**
   - Sudo permissions allowing command execution as root without password
   - Dangerous binary permissions (ZIP with command injection capability)

4. **🛡️ Inadequate Security Controls**
   - Lack of network segmentation for sensitive services
   - No file integrity monitoring
   - Insufficient logging and monitoring

### 🛠️ Security Recommendations

**🚨 Immediate Actions:**
1. Update Apache Tomcat to the latest version
2. Review and restrict sudo permissions for all users
3. Change all user passwords using strong, unique credentials
4. Implement network segmentation for Tomcat services

**📈 Long-term Improvements:**
1. **🔐 Access Control**
   - Regular sudo permissions audit
   - Principle of least privilege implementation
   - Multi-factor authentication for administrative access

2. **📊 Monitoring & Logging**
   - Implement intrusion detection systems
   - Enable comprehensive logging for Tomcat and system access
   - Regular security audits and vulnerability assessments

3. **🔒 Security Hardening**
   - Regular security patching schedule
   - Application whitelisting where possible
   - File integrity monitoring implementation

### 🛡️ Defense-in-Depth Strategies
1. Web Application Firewall (WAF) implementation
2. Regular security awareness training
3. Incident response plan development
4. Continuous security monitoring

---
**📝 Report Generated By**: Cybersecurity Student 🎓  
**📅 Date**: September 26, 2025  
**🎯 Purpose**: Educational penetration testing exercise  
**⚠️ Note**: This report contains information about security vulnerabilities for educational purposes only.
