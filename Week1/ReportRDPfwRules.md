# Lab Report: Firewall Policy Implementation & Traffic Verification

**Date:** 2026-06-030  
**Firewall Platform:** OPNsense 26.1.6\
**Change Authority/Ticket:** Lab Requirements

## 1. Objective & Business Justification
### - **Objective:** Allow RDP from Kali to Windows 11
- **Justification:** It's required to apply monitoring, analyse and even exploit

## 2. Network Topology & Matrix
* review LabTopo.png


* INNET\_SEC:

    	+ Kali Linux --ipv4 (10.10.2.114)

    	+ Wazuh server --ipv4 (10.10.2.173)

* INNET\_PROD:

    	+ Windows 11 family edition --ipv4 (10.10.3.101)

    	+ Windows 11 Pro --ipv4 (10.10.3.199)

    	+ Ubuntu server --ipv4 (10.10.3.198)

* OPNsense:

    	+ LAN "em1" --ipv4 (10.10.2.3/24)

    	+ OPT1 "em2" --ipv4 (10.10.3.3/24)

    	+ WAN "em0" --ipv4 (DHCP)




### 2.1 Zone Definitions
- **Source Zone:** [ INNET_SEC (10.10.2.0/24)]
- **Destination Zone:** [INNET_PROD (10.10.3.0/24)]

### 2.2 Intended Traffic Matrix

| Source IP/Subnet | Destination IP/Subnet | Protocol | Port | Action | Purpose |
|:-----------------|:----------------------| :--- | :--- | :--- | :--- |
| 10.10.2.114/32   | 10.10.3.199/32        | TCP | 3389 | ALLOW | Management RDP Access |
| 10.10.2.114/32   |  10.10.3.0/24       | ANY | ANY | REJECT | Default Inter-Zone Block |

## 3. Implementation Steps (Rule Configuration)
Detailed chronological steps of how you configured the rules in the GUI or CLI.

### 3.1 Rule Creation
1. Navigated to **Firewall -> Rules -> [LAN]**.
2. Clicked **Add/Insert** to create a new rule.
3. Configured the following parameters:
    - **Action:** Pass
    - **Protocol:** TCP
    - **Direction**: In
    - **Source:** [10.10.2.114/32]
    - **Destination:** [10.10.3.199/32]
    - **Destination Port Range:** [3389 to 3389]
    - **Log:** [X] Enable logging for this rule *(Crucial for SIEM/Wazuh verification!)*
    - **Save**
4. **Firewall -> Rules -> [OPT1]** Clicked **Add/Insert** to create a new rule.
5. Configured the following parameters:
    - **Action:** Pass
    - **Protocol:** TCP
    - **Direction**: Out
    - **Source:** [10.10.2.114/32]
    - **Destination:** [10.10.3.199/32]
    - **Destination Port Range:** [3389 to 3389]
    - **Log:** [X] Enable logging for this rule
    - **Save**
    - **Click Apply changes**
### 3.2 Policy Order Validation
- Explain where the rule sits in the firewall rule list ("Placed above the implicit 'Deny All' rule at the bottom of the interface stack").
- Disable RDP from Floating interface in the same manner

## 4. Verification & Testing (The "Proof")

### 4.1 Positive Testing (Traffic Allowed)
- **Action:** Executed connection command from Kali.
    - **Command:**  
      "xfreerdp /v:10.10.3.199 /u:vboxuser2 /p:1234"
      "y"
- **Evidence:**
* review Obj2.2D3.png

### 4.2 Negative Testing (Traffic Blocked)
- **Action:** Attempted unauthorized connection to verify the firewall blocks everything else.
    - **Command:** "ping 10.10.3.198" .
        - **Evidence:**
          ##     
              (root@kali)-[/home/kali/Desktop]
              # ssh wazuh-user@10.10.3.177
              ssh: connect to host 10.10.3.177 port 22: Connection refused
            
              (root@ kali)-[/home/kali/Desktop]
              # ssh kali@10.10.3.198
              kalia10.10.3.198's password:
            
              (root@ kali)-[/home/kali/Desktop]
              # ssh wazuh-user@10.10.3.177
              ssh: connect to host 10.10.3.177 port 22: Connection refused
            
              (root@kali)-[/home/kali/Desktop]
              # ping 10.10.3.198
              PING 10.10.3.198 (10.10.3.198) 56(84) bytes of data.
              ^c
              --- 10.10.3.198 ping statistics ---
              3 packets transmitted, 0 received, 100% packet loss, time 2051ms
            
              (root@kali)-[/home/kali/Desktop]
              # ping 10.10.3.198
              PING 10.10.3.198 (10.10.3.198) 56(84) bytes of data.
              ^C
              --- 10.10.3.198 ping statistics ---
              3 packets transmitted, 0 received, 100% packet loss, time 2029ms
            
              (root@ kali)-[/home/kali/Desktop]
              # ping 10.10.3.177
              PING 10.10.3.177 (10.10.3.177) 56(84) bytes of data.
              ^C
              --- 10.10.3.177 ping statistics ---
              4 packets transmitted, 0 received, 100% packet loss, time 3048ms]
          ##

## 5. Firewall & SIEM Log Observations
Provide the security analyst view. What do the logs look like when the rule triggers?

### 5.1 Live Firewall Log Capture
```text
[pass   LAN tcp     10.10.2.144:48392 -> 10.10.3.199:3389]
[pass   OPT tcp     10.10.2.144:48392 -> 10.10.3.199:3389]
[block  LAN icmp    10.10.2.144       -> 10.10.3.177]

```

### 5.2 SIEM / Wazuh Alert

    * timestamp: 2026-06-08T01:47:33
      * action: [block]
      * dir: [in]
      * interface: em1
      * reason: match
      * ipversion: 4
      * ipflags: DF
      * length: 60
      * offset: 0
      * protoname: tcp
      * protonum: 6
      * id: 2085
      * sr (src ip): 10.10.2.114
      * srchostname: 10.10.2.114
      * dst: 10.10.3.177
      * dsthostname: 10.10.3.177
      * dstport: 22 (SSH)
      * seq: 3880431445
      * ack: 116
      * anchorname: Q ae0bc8451bda11d06b12a1013ad37797
      * datalen: 0
      * ecn: Non spécifié / Vide
      * label: Non spécifié / Vide
      * rid: Non spécifié / Vide
      * rulenr: Non spécifié / Vide


## 6. Security Analysis & Hardening Recommendations
- **Risk Assessment:** Does keeping this port open introduce risk? ( "Leaving RDP open across segments exposes the Windows host to lateral brute-force attacks if Kali is compromised.")
- **Mitigation:** How can you secure it further? ( "Implement Multi-Factor Authentication on the target, restrict access to a specific MAC address, or use a Jump box / Bastion host instead.")
## 7. Conclusion & References
- Allow RDP from Kali to Windows 11 Done!
- Windows 11 Pro: https://www.microsoft.com/en-us/software-download/windows11
- OPNsense rule: https://docs.opnsense.org/manual/firewall.html
- Wazuh Windows agent: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-windows.html
