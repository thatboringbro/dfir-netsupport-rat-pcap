# Incident Investigation Report: NetSupport Manager RAT Compromise

## Case Overview
* **Case ID:** `INC-2026-0228A`
* **Incident Date:** February 28, 2026 
* **Target Domain:** `easyas123.tech` (AD Environment: `EASYAS123`) 
* **Classification:** CONFIDENTIAL // INTERNAL USE ONLY 
* **Investigated By:** Security Operations Center (SOC) 

---

## 1.0 Executive Summary
On February 28, 2026, at 19:55 UTC, a local Windows workstation within the `EASYAS123` Active Directory domain segment was compromised via a NetSupport Manager Remote Access Trojan (RAT) infection. The primary breach vector was identified as persistent outbound communication targeting a malicious Command and Control (C2) node located at external IP address `45.131.214[.]85` over TCP port 443. 

The affected host asset was isolated from the local broadcast domain to mitigate the risk of internal lateral movement toward the Active Directory Domain Controller. Remediation pipelines, including credential eviction, have been executed. No structured data exfiltration has been definitively verified from network layer logs; however, advanced host-based endpoint forensics are currently underway.

---

## 2.0 Affected Assets & Scope
The operational boundaries and specific system identity metrics verified during token triage are detailed below:

| Asset Attribute | Value Captured From Investigation |
| :--- | :--- |
| **IP Address** | `10.2.28[.]88`  |
| **MAC Address** | `00:19:d1:b2:4d:ad`  |
| **Host Name** | `DESKTOP-TEYQ2NR`  |
| **User Account Name** | `brolf`  |
| **User Full Name** | `Becka Rolf`  |
| **Active Directory Domain** | `easyas123[.]tech`  |
| **Operating System Platform**| `Microsoft Windows Client`  |

---

## 3.0 Technical Analysis & Evasion Narrative

### Initial Detection & Telemetry
The Security Information and Event Management (SIEM) system registered multiple signature triggers correlating to NetSupport Manager RAT behaviors from external node `45.131.214[.]85` over TCP port 443 beginning exactly at 19:55 UTC.

### Network Tunneling Verification
Packet analysis revealed that the threat actor utilized port 443 to tunnel cleartext HTTP communications inside what appeared superficially to be standard secure traffic (HTTPS). This evasion technique is specifically crafted to masquerade command-and-control operations, successfully bypassing perimeter deep packet inspection and standard protocol validations.

![Network Tunneling Evidence](./screenshots/pcap-nesupport-traffic.png)
*Figure 1: Wireshark packet stream mapping cleartext HTTP requests traveling across standard secure port 443.*

### Identity Discovery
Deep analysis of internal Active Directory protocol traffic crossing the segment interface successfully extracted local domain session properties. Security Account Manager Remote Protocol (SAMR) query tracking bypassed protocol limitations, mapping the compromise directly to local domain user account `brolf` (Becka Rolf).

![Identity Discovery Evidence](./screenshots/pcap-full-name-wireshark.png), ![Username Discovery Evidence](./screenshots/pcap-username-wireshark.png)
*Figure 2: Extraction of SAMR `QueryUserInfo` response mappings displaying the compromised account identity credentials.*

---

## 4.0 MITRE ATT&CK Mapping
This incident demonstrates the following attacker tactics and techniques tracked within the enterprise matrix:

| Tactic | Technique | Description |
| :--- | :--- | :--- |
| **Execution** | `T1059.001` (PowerShell) | Command execution initiated via RAT platform. |
| **Defense Evasion** | `T1140` (Deobfuscate/Decode Files) | Cleartext HTTP tunneled through standard HTTPS infrastructure. |
| **Command & Control**| `T1071.001` (Web Protocols) | Command and Control communications routed over HTTP/443. |
| **Exfiltration** | `T1041` (Exfiltration Over C2) | Potential user and process data exfiltration over active C2 channels. |

---

## 5.0 Indicators of Compromise (IOCs)
The following network indicators were extracted during the incident lifecycle and should be ingested immediately into perimeter defenses:

| Network/Host Indicator | Threat Context / Metric | Operational Directive |
| :--- | :--- | :--- |
| `45.131.214[.]85` | Malicious C2 Infrastructure Node | Block at Edge Firewall. |
| `hxxp://45.131.214[.]85/fakeurl.htm` | Malicious Active Delivery URI | Blacklist on Proxy Gateway. |

---

## 6.0 Remediation & Hardening Actions
1. **Host Quarantine:** Maintain absolute network isolation of workstation `10.2.28[.]88` from the local broadcast network segment to eliminate internal lateral compromise vectors.
2. **Perimeter Enforcement:** Apply immediate, centralized firewall rules blocking inbound and outbound traffic to the threat infrastructure listed in Section 5.0.
3. **Identity Eviction:** Execute a forced global Active Directory password reset for user identity `brolf` and invalidate all active Kerberos Ticket Granting Tickets (TGT) tied to the account.
4. **Defensive Engineering Evaluation:** Pivot analysis to endpoint forensics (Sysmon event analysis and Master File Table tracking) to audit the potential local execution of commands during periods of unparsed network anomalies.

If you have any questions or want to reach out, please contact me on X (formerly twitter) : [@thatboringbro (redacted)](https://x.com/thatboringbro)
