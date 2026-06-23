# 	(Incident Summary Report)







_________________________________________________________________________________________________________________________________

## INCIDENT SUMMARY REPORT

Incident ID: INC-2026-0894

Date of Report: June 15, 2026

Incident Commander: Koussay Ftita (Security Analyst)

Classification / Severity: High Severity / Unauthorized Access (Brute-Force)

__________________________________________________________________________________________________________________________________________________

## 1. Executive Summary

> #### On June 15, 2026, between 02:03 and 02:22 UTC, an internal threat actor executed sequential, high-volume brute-force attacks targeting critical remote access services across our network infrastructure.
_________________________________________________________________________________________________________________________________________________
>>* #### SSH Compromise (Ubuntu Server): At 02:18 UTC, the attacker successfully compromised a local account (kali@10.10.3.198) via SSH. The service lacked robust login hardening and rate-limiting policies. The attacker maintained unauthorized access for approximately 15 minutes, utilizing the compromised server to conduct internal network reconnaissance and scanning.
>>* #### RDP Compromise (Windows 11 Desktop): At 02:22 UTC, leveraging insights from the initial network scan, the attacker launched a secondary brute-force attack against the Remote Desktop Protocol (RDP) service on a Windows 11 workstation, successfully compromising the account vboxuser2@10.10.3.199.
___________________________________________________

> #### The incident was detected by the SIEM at 02:57 UTC due to abnormal internal scanning signatures. The Incident Response team successfully contained the threat by 03:05 UTC by terminating all active unauthorized sessions and isolating the affected hosts. Forensic verification confirms that the threat has been completely eradicated, no persistent backdoors were established, and no data exfiltration occurred. All systems have been restored to 100% operational status.

## 2. Incident Timeline (UTC)



* #### 02:03:32 – Attack begins. High-volume, distributed credential-stuffing/brute-force login attempts hit the SSH Service on UBUNTU Server.

* #### 02:18:29 – Successful login achieved on account kali@10.10.3.198 from IP 10.10.2.114.

* #### 22:18:10 – Attacker initiates internal network scanning (RDP/SSH mapping) from the compromised endpoint (UBUNTU Server).

* #### 02:22:21 – Attack begins. High-volume, distributed credential-stuffing/brute-force login attempts hit the RDP Service on Windows 11 Pro (Desktop).

* #### 02:57:47 – *[Detection]* SIEM (Wazuh) triggers a High-Severity alert for abnormal lateral movement and internal scanning.

* #### 03:05:40 – *[Containment]* IR Team revokes the user's active SSH && RDP sessions and disables the compromised Active Directory account.

* #### 03:30:00 – *[Eradication \& Recovery]* Host isolation completed; forensic verification confirms zero persistent backdoors.



## 3. Technical Analysis \& Scope



* **Root Cause:** The compromised server was mistakenly connected to external network without any security settings like HTTPS or hardening login process and that leaks an attack vector bypassed global conditional access policies requiring MFA.

* **Attack Vector:** targeted Brute-Force / Credential Stuffing using only one IP address.

* **Impacted Infrastructure:** 2 Virtual Endpoints (Main Server && User Workstation), 2 Remote Sessions log.

* **Indicators of Compromise (IOCs):** Successful Login attempt after many failed attempts.

* **Primary Attacker IP:** 10.10.2.114 (Hosting Provider).

* **User-Agent string:** Mozilla/5.0 (Kali Linux; CLI; x64; CustomBot/2.1)

* **Malicious Activity:** Network scanning tool signature nmap detected on internal subnet 10.10.3.0/24.



## 4. Containment \& Remediation Actions:



>* Immediate Isolation: Disabled the compromised user identity in Microsoft Entry ID / Active Directory.

>* Eradication: Ran full EDR (Endpoint Detection and Response) deep scans on the user's assigned physical laptop, Server and virtual environments. Cleared malicious volatile memory artifacts.

>* Network Hardening: Implemented a strict rate-limiting rule on the perimeter firewall to block IPs after 5 failed authentication attempts within 5 minutes, regardless of targeted patterns.



## 5. Preventative Actions \& Lessons Learned:



1. MFA Audit (Immediate): Run a script to identify any other active session currently excluded from Remote connection. (Owner: Process ID | Deadline: 24 Hours).

2. SIEM Tuning (30 Days): Update SIEM logic to trigger alerts on targeted authentication failures targeting the same username before a successful login occurs. (Owner: SOC Team | Deadline: July 15, 2026).



________________________________________________________________________________________________________________________________________________________________________________________________________________________________





