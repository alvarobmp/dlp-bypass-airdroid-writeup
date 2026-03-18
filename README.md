# 🔐 DLP Bypass via Unmanaged WiFi Hotspot + AirDroid

> **Security Research | Endpoint Security | Data Exfiltration**  
> A real-world DLP policy gap found in a Windows 10 corporate endpoint with multiple active security controls.

---

## 📋 Overview

This repository documents a **Data Loss Prevention (DLP) bypass** technique identified in a corporate endpoint environment running:

- ✅ BitLocker Full Disk Encryption
- ✅ BitLocker To Go on removable storage
- ✅ Cisco AnyConnect Full Tunnel VPN
- ✅ Restricted standard user (no admin rights)

**Despite all three controls being active**, a confidential file was successfully exfiltrated using a personal WiFi hotspot and AirDroid Personal — a free consumer app — in under 5 minutes, with no malware, no exploits, and no privilege escalation.

---

## 🎯 Who should read this

| Audience | Why it matters |
|---|---|
| **SME / PYME owners** | Your security investment may have a blind spot |
| **IT administrators** | Policy stacking ≠ full coverage |
| **Security consultants** | Reproducible case for endpoint DLP audits |
| **SOC analysts** | This vector leaves no trace in standard Windows event logs |

---

## 🗂️ Repository Contents

```
📄 README.md                        ← This file
📄 dlp-bypass-airdroid-writeup.md   ← Full technical writeup
```

---

## ⚡ Quick Summary

```
Target:   Windows 10 laptop — Lenovo 2025
Controls: BitLocker FDE + BitLocker To Go + Cisco AnyConnect Full Tunnel
Vector:   Personal WiFi hotspot → AirDroid HTTP server (port 8888)
Result:   .docx file exfiltrated in plaintext — all DLP controls bypassed
MITRE:    T1011 · T1048
```

---

## 🔍 Key Finding

The organization implemented **three independent security controls**, each targeting a specific threat:

| Control | Threat it addresses | Bypassed? |
|---|---|---|
| BitLocker To Go | USB exfiltration | ✅ Yes — AirDroid transfers files in plaintext over WiFi |
| Cisco AnyConnect Full Tunnel | Internet-based exfiltration | ✅ Yes — local WiFi traffic is not routed through VPN |
| BitLocker FDE | Physical device theft | ✅ Yes — operates inside the authenticated session |

**The gap:** None of the controls monitored or restricted local wireless traffic between the endpoint and a personal mobile hotspot.

---

## 🛡️ Recommended Remediation

1. **Network Access Control (NAC)** — Block connections to unauthorized WiFi networks including personal hotspots
2. **MDM Policy** — Restrict endpoint from joining non-approved SSIDs via Microsoft Intune or Jamf
3. **Host-based Firewall Rule** — Block outbound TCP 8888 to RFC 1918 address ranges
4. **Full DLP Solution** — Replace ad-hoc policy stacking with content-aware DLP (Microsoft Purview, Forcepoint)

---

## 📖 Full Writeup

👉 [Read the complete technical writeup](./dlp-bypass-airdroid-writeup.md)

Includes: lab environment, reconnaissance methodology, step-by-step reproduction, MITRE ATT&CK classification, impact assessment, and detailed remediation guidance.

---

## ⚠️ Disclaimer

This research was conducted in a **controlled lab environment** for educational and defensive security purposes only. All techniques described should only be performed on systems for which **explicit written authorization** has been obtained. The author does not condone unauthorized access to any computer system or network.

---

## 👤 Author

**Alvaro Martinez** | IT Infrastructure Specialist  
Google Cybersecurity · Cisco CyberOps · ~10 years experience  

[![GitHub](https://img.shields.io/badge/GitHub-alvarobmp-181717?style=flat&logo=github)](https://github.com/alvarobmp)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-alvaromrtnz-0A66C2?style=flat&logo=linkedin)](https://www.linkedin.com/in/alvaromrtnz/)

---

*If this writeup was useful for your security work, consider leaving a ⭐ on the repository.*
