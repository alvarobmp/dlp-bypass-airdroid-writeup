# DLP Bypass via Unmanaged WiFi Hotspot + AirDroid HTTP Transfer

**Author:** Alvaro R. | IT Infrastructure Specialist  
**Certifications:** Google Cybersecurity · Cisco CyberOps  
**Experience:** ~10 years in IT | 4 years in Infrastructure Administration  
**GitHub:** [github.com/alvarobmp](https://github.com/alvarobmp) · **LinkedIn:** [linkedin.com/in/alvaromrtnz](https://www.linkedin.com/in/alvaromrtnz/)  
**Date:** March 2026  
**Category:** Endpoint Security · DLP · Data Exfiltration  
**Difficulty:** Medium  

---

## 1. Executive Summary

This writeup documents a **Data Loss Prevention (DLP) policy bypass** identified in a simulated corporate endpoint environment. A Windows 10 laptop configured with USB device blocking, Cisco AnyConnect Full Tunnel VPN enforcement, and restricted user privileges was found to be vulnerable to local file exfiltration via an unmanaged WiFi hotspot and the AirDroid Personal web interface (port 8888).

The scenario demonstrates that organizations which focus exclusively on blocking traditional exfiltration channels (USB, corporate internet) while leaving local network access uncontrolled remain exposed to a functional and low-complexity data exfiltration vector.

**Result:** A confidential Word document was successfully transferred from the restricted endpoint to an external device without admin privileges, without USB access, and without corporate internet connectivity.

---

## 2. Lab Environment

> *This scenario was reproduced in a controlled lab environment replicating common SME (Small and Medium Enterprise) corporate endpoint configurations observed in Lima, Peru.*

| Component | Detail |
|---|---|
| Endpoint | Lenovo laptop, 2025 model |
| Operating System | Windows 10 |
| VPN Client | Cisco AnyConnect Secure Mobility Client |
| VPN Policy | Full Tunnel (all traffic routed through VPN) |
| USB Policy | Device Control enabled — removable storage blocked |
| User Privileges | Standard user (no local admin rights) |
| Mobile Device | Android smartphone with AirDroid Personal (free) |

---

## 3. Reconnaissance — Identifying Active Controls

Before attempting any transfer, the following controls were confirmed active on the endpoint:

**3.1 USB Block Confirmed**
A USB flash drive was connected to the laptop. The operating system did not mount the device. No drive letter was assigned. No notification appeared. This confirmed an active **Device Control policy** blocking removable storage.

**3.2 VPN Full Tunnel Confirmed**
Cisco AnyConnect was opened. Authentication credentials were entered. The client returned:

```
Authentication failed.
```

Without an active VPN session, no internet access was available — including over an external WiFi network. This confirmed a **Full Tunnel policy**: all traffic is routed through the VPN gateway, and without it, the endpoint has no external connectivity regardless of the underlying network.

**3.3 Admin Privileges Confirmed Absent**
No attempts to modify system settings, install software, or interact with `services.msc` in a privileged context were made — consistent with a standard user account.

---

## 4. Vulnerability Identified — The Policy Gap

### What was controlled:
- ✅ USB / Removable storage → Blocked via Device Control
- ✅ Internet access → Blocked via Cisco AnyConnect Full Tunnel
- ✅ Local administrator access → Restricted

### What was NOT controlled:
- ❌ Connection to arbitrary external WiFi networks (personal hotspot)
- ❌ Local network traffic on non-corporate WiFi
- ❌ Use of AirDroid or equivalent local HTTP servers on mobile devices

### The gap:
The security policy treated **internet access** and **data exfiltration** as the same problem. They are not. A device connected to a personal hotspot has no corporate internet access — but it does have **local IP connectivity** between the laptop and the mobile phone. This local channel was completely unmonitored and uncontrolled.

### MITRE ATT\&CK Classification:

| Field | Value |
|---|---|
| **Tactic** | Exfiltration (TA0010) |
| **Technique** | T1011 — Exfiltration Over Other Network Medium |
| **Sub-technique** | T1048 — Exfiltration Over Alternative Protocol |
| **CWE** | CWE-284: Improper Access Control |

---

## 5. Exploitation — Step-by-Step Reproduction

> ⚠️ **This section is provided for educational and defensive purposes only. Reproducing this technique on any system without explicit written authorization from the asset owner is illegal.**

### Prerequisites
- Android smartphone with [AirDroid Personal](https://www.airdroid.com/) installed (free)
- Target laptop connected to the phone's WiFi hotspot

### Step 1 — Enable mobile hotspot
On the Android device, activate the personal WiFi hotspot. The laptop connects to this network. At this point the laptop has **no internet** (VPN is down) but has **local IP connectivity** to the phone.

### Step 2 — Launch AirDroid web interface
On the Android device, open AirDroid Personal. Navigate to **AirDroid Web** and note the local IP address displayed. In this scenario the assigned IP was:

```
192.168.43.1
```

AirDroid exposes a local HTTP server on **port 8888** accessible to any device on the same network.

### Step 3 — Access AirDroid from the restricted laptop
On the laptop, open any web browser (no internet required — this is a local request). Navigate to:

```
http://192.168.43.1:8888
```

The AirDroid web interface loads successfully in the browser.

### Step 4 — Transfer the file
From the laptop's file explorer, the target `.docx` file was located at:

```
C:\Users\[username]\Documents\[filename].docx
```

The file was dragged and dropped into the AirDroid web interface. Transfer completed successfully.

### Step 5 — File retrieved on mobile device
The document was received on the Android device via AirDroid and was immediately available for forwarding via email, messaging, or cloud storage — with no VPN, no USB, and no admin privileges required.

---

## 6. Impact Assessment

| Risk | Description |
|---|---|
| **Confidential data exfiltration** | Any file accessible to the standard user can be extracted |
| **Policy bypass** | All three DLP controls (USB, VPN, admin) rendered ineffective |
| **No audit trail** | Standard Windows event logs do not capture AirDroid HTTP transfers |
| **Low skill requirement** | AirDroid is free, widely available, requires no technical knowledge |
| **Scalable** | Applicable to any endpoint with the same policy configuration |

---

## 7. Recommended Remediation

Organizations deploying endpoint security policies should address this vector through a layered approach:

### 7.1 Network Access Control (NAC)
Implement **802.1X authentication** or a NAC solution that prevents the endpoint from connecting to unauthorized WiFi networks, including personal hotspots. This closes the initial channel.

> Tools: Cisco ISE, Aruba ClearPass, open-source PacketFence

### 7.2 Mobile Device Management (MDM) on personal hotspot policy
If corporate-managed devices are in scope, deploy an MDM policy that restricts the endpoint from joining non-approved SSIDs.

> Tools: Microsoft Intune, Jamf

### 7.3 Application Whitelisting on endpoint
Block execution of browser-based file transfers to local IPs outside the corporate subnet. A host-based firewall rule blocking outbound connections to `192.168.x.x:8888` (and equivalent AirDroid ports) would prevent this specific vector.

```
Block outbound TCP 8888 to 192.168.0.0/16
Block outbound TCP 8888 to 172.16.0.0/12
Block outbound TCP 8888 to 10.0.0.0/8
```

### 7.4 Full DLP solution
Replace ad-hoc policy stacking with a dedicated DLP solution that monitors **content movement** regardless of channel.

> Tools: Symantec DLP, Microsoft Purview Information Protection, Forcepoint

---

## 8. Lessons Learned

**For IT administrators and SME owners:**

Blocking USB ports and enforcing VPN are necessary but not sufficient. A complete DLP strategy must account for all possible data channels — including local wireless transfers. The question is not *"did we block the obvious paths?"* but *"what paths did we forget to consider?"*

**For security consultants:**

This vector is reproducible in minutes with freely available consumer software. It requires no malware, no exploits, and no privilege escalation. The attack surface exists in policy design, not in software vulnerabilities — which makes it invisible to most vulnerability scanners.

---

## 9. References

- [MITRE ATT&CK T1011 — Exfiltration Over Other Network Medium](https://attack.mitre.org/techniques/T1011/)
- [MITRE ATT&CK T1048 — Exfiltration Over Alternative Protocol](https://attack.mitre.org/techniques/T1048/)
- [CWE-284: Improper Access Control](https://cwe.mitre.org/data/definitions/284.html)
- [AirDroid Official Documentation](https://help.airdroid.com/)
- [Cisco AnyConnect Administrator Guide](https://www.cisco.com/c/en/us/support/security/anyconnect-secure-mobility-client/products-installation-and-configuration-guides-list.html)
- [Microsoft Intune Device Compliance Policies](https://learn.microsoft.com/en-us/mem/intune/protect/device-compliance-get-started)

---

## 10. Disclaimer

This writeup was produced for **educational and defensive security purposes only**. The scenario described was reproduced in a controlled lab environment. The author does not condone or encourage unauthorized access to any computer system or network. All techniques described should only be performed on systems for which explicit written authorization has been obtained.

---

*Alvaro R. | IT Infrastructure Specialist | Google Cybersecurity · Cisco CyberOps*  
*[github.com/alvarobmp](https://github.com/alvarobmp) · [linkedin.com/in/alvaromrtnz](https://www.linkedin.com/in/alvaromrtnz/)*
