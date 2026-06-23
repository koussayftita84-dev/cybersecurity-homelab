# Lab Report: Firewall Policy Implementation & Traffic Verification

**Date:** 2026-06-030  
**Firewall Platform:** OPNsense 26.1.6\
**Change Authority/Ticket:** Lab Requirements

## 1. Objective & Business Justification
### - **Objective:** Allow SSH from Kali to Ubuntu Server 
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
| :---             | :---                  | :---     | :--- | :---   | :---    |
| 10.10.2.114      | 10.10.3.198           | TCP      | 22   | ALLOW  | Management SSH Access |
| 10.10.2.114    | 10.10.3.0/24          | TCP      | 22   | REJECT | Default SSH Reject |

## 3. Implementation Steps (Rule Configuration)
Detailed chronological steps of how you configured the rules in the GUI or CLI.

### 3.1 Rule Creation
1. Navigated to **Firewall -> Rules -> [LAN]**.
2. Clicked **Add/Insert** to create a new rule.
3. Configured the following parameters:
   - **Action:** Pass 
   - **Direction** In
   - **Protocol:** TCP 
   - **Source:** [10.10.2.114/32]
   - **Destination:** [10.10.3.198/32]
   - **Destination Port Range:** [ 22 to 22 ]
   - **Log:** [X] Enable logging for this rule *(Crucial for SIEM/Wazuh verification!)*
4. Navigated to **Firewall -> Rules -> [OPT1]**.
5. Clicked **Add/Insert** to create a new rule and configured:
   - **Action:** pass
   - **Direction** out 
   - **Protocol:** TCP/UDP
   - **Source:** [10.10.2.114/32]
   - **Destination:** [10.10.3.198/32]
   - **Destination Port Range:** [ 22 to 22 ]
   - **Log:** [X] Enable logging for this rule 
6. Clicked **Apply changes**
### 3.2 Policy Order Validation
- Explain where the rule sits in the firewall rule list ("Placed above the implicit 'Default SSH Block' rule at the bottom of the interface stack").
- SSH checkout rule block any ssh attempts by default 
## 4. Verification & Testing (The "Proof")

### 4.1 Positive Testing (Traffic Allowed)
- **Action:** Executed connection command from source host.
- **Command:** "ssh kali@10.10.3.198" 
  - **Evidence:** 
    ```text
    [
    (root@kali)-[/home/kali]# ssh kalig10.10.3.198
    kali@10.10.3.198's password:
    Welcome to Ubuntu 26.04 LTS (GNU/Linux 7.0.0-14-generic x86_64)
    * Documentation:  https://docs.ubuntu.com
    * Management:     https://landscape.canonical.com
    * Support:        https://ubuntu.com/pro
    System information as of Sat May 9 03:32:35 PM UTC 2026

    System load: 0.01                      Processes: 208
    Usage of /: 27.4% of 12.33GB           Users logged in: 0
    Memory usage: 27%                      IPv4 address for enp0s8: 10.10.3.198
    Swap usage: 0%

    Expanded Security Maintenance for Applications is not enabled.

    25 updates can be applied immediately.
    10 of these updates are standard security updates.
    To see these additional updates run: apt list -- upgradable

    Enable ESM Apps to receive additional future security updates.
    See https://ubuntu.com/esm or run: sudo pro status

    Failed to connect to https://changelogs.ubuntu.com/meta-release. Check your Internet connection or proxy settings

    Last login: Sat May 9 14:26:40 2026 from 10.10.3.101
    ] 
    ```

### 4.2 Negative Testing (Traffic Blocked)
- **Action:** Attempted unauthorized connection to verify the firewall blocks everything else.
- **Command:** `"ssh wazuh-user@10.10.3.177".
- **Evidence:** 
  ```text
  [       (root@kali)-[/home/kali/Desktop]
          # ssh root@10.10.3.3
          ^c

          (root@kali)-[/home/kali/Desktop]
          # ssh wazuh-user@10.10.3.177
          ^C

          (root@kali)-[/home/kali/Desktop]
          # ping 10.10.3.198
          PING 10.10.3.198 (10.10.3.198) 56(84) bytes of data.
          64 bytes from 10.10.3.198: icmp_seq=1 ttl=63 time=3.97 ms
          64 bytes from 10.10.3.198: icmp_seq=2 ttl=63 time=12.0 ms
          ^C
          --10.10.3.198 ping statistics--
          2 packets transmitted, 2 received, 0% packet loss, time 1014ms
          rtt min/avg/max/mdev = 3.972/7.986/12.001/4.014 ms
  ]
  ```

## 5. Firewall & SIEM Log Observations
Provide the security analyst view. What do the logs look like when the rule triggers?

### 5.1 Live Firewall Log Capture
Paste the raw log line from your firewall console showing the match:
```text
  [
    + OPT1  Out 2026-06-07T23:27:25 TCP  10.10.2.114:41986 10.10.3.198:22  pass Allow SSH from Kali to Ubuntu Server
    + LAN   In  2026-06-07T23:27:25 TCP  10.10.2.114:41986 10.10.3.198:22  pass
    + LAN   In  2026-06-07T23:27:16 TCP  10.10.2.114:34012 10.10.3.177:22  block Default deny / state violation rule
   ]
```

### 5.2 SIEM / Wazuh Alert 
- JSON 
```

        "_index": "wazuh-alerts-4.x-2026.06.07",
        "_id": "JhFhpJ4BEGhNQ2hZAONI",
        "_version": 1,
        '_score": null,
        _source":
        "predecoder": {
        'hostname": "Serveur",
        "program_name": "sshd-session"
        "timestamp": "May 10 03:05:40"
        },
        "input": {
        "type": "log"
        },
        "agent": {
        "ip": "10.10.3.198"
        name": "ubuntu_server"
        "id": "005"
        },
        "manager": {
        "name": "wazuh-server"
        },
        'data": {
        "srcip": "10.10.2.114",
        "dstuser": "kali",
        "srcport": "41986"
        },
        "rule": {
        "mail": false,
        "level": 3,
        "hipaa": [
        "164.312.b"
        ],
        "pci_dss": [
        "10.2.5"

        ],

        "tsc": [
        "CC6.8"
        "CC7.2",
        "CC7.3"
        ]
        "description": "sshd: authentication success.",
        "groups": [
        "syslog",
        "sshd",
        "authentication_success"

        "nist_800_53": [
        "AU. 14",
        "AC.7"
        ]
        ],

        gdpr": [
        "IV_32.2"
        ]
        "firedtimes": 2,
        "mitre": {
        "technique": [
        "Valid Accounts",
        "Remote Services"
        ],
        "id": [
        "T1078"
        "T1021"

        ],
        "tactic": [
        "Defense Evasion",
        "Persistence",
        "Privilege Escalation",
        Privitege Escatatlon 1
        "Initial Access",
        "Lateral Movement"
        ],
        "id": "5715",
        gpg13": [
        "7.1",
        "7.2"
        ],
        "location": "journald",
        "decoder": {
        'parent": "sshd",
        "name": "sshd"
        },
        "id": "1780874270.380555",
        "full_log": "May 10 03:05:40 Serveur sshd-session[31810]: Accepted password for kali
        from 10.10.2.114 port 41986 ssh2",
        "timestamp": "2026-06-07T23:17:50.560+0000"
        },
        "fields": {
        "timestamp": [
        "2026-06-07T23:17:50.560Z"
        ]
        },
        "highlight": {
        "data.srcip": [
        "@opensearch-dashboards-highlighted-field@10.10.2.114a/opensearch-dashboards-
        highlighted-fielda"
        ]
        },
        "full_log": [
        'May 10 03:05:40 Serveur sshd-session[31810]: Accepted password for kali from
        @opensearch-dashboards-highlighted-fielda10.10.2.114a/opensearch-dashboards-highlighted-
        fielda port 41986 ssh2"
        ]
        },
        "sort": [
        1780874270560
        ]
  ```
## 6. Security Analysis & Hardening Recommendations
- **Risk Assessment:** Does keeping this port open introduce risk? ("Leaving this port open can be vulnerable, it can be exploited by Bruteforce attack")
- **Mitigation:** How can you secure it further? 
("Configure Security statement that block any ip or host trying attempts more than 10 time")

## 7. Conclusion & References
- Allow SSH from Kali to Ubuntu Server Done!
- OPNsense rule: https://docs.opnsense.org/manual/firewall.html
- Wazuh Windows agent: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-windows.html
