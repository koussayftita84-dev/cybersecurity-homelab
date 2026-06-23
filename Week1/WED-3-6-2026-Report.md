#     	Report

* ## Objective:

#### 1) OPNsense:
  +   Create an allow rule for SSH from Kali to Ubuntu Server
  +   Create an allow rule for RDP from Kali to Win 11
#### 2) Kali:
  +   SSH into Ubuntu Server, then check OPNsense live log
  +   RDP into Win 11, then check OPNsense live log
#### 3) Logs Review:
  * Ubuntu Server:
    + View "/var/log/auth.log"
    + find your successful login
  * Windows 11:
    +   Check Security logs (Successful 4624), (Failed 4625)
  *  Deploy Wazuh agent
#### 4) Python:
  +   Write a function to count failed logins per IP from "auth.log" 
  +   Write a function to count failed logins per IP from exported Security log CSV
#### 5) Report Report: Document the SSH and RDP allow rule and log evidence



* ## Lab Setup/Topology:
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



* ## Step Performed:


* ### Creating rules in OPNsense :

  * SSH from Kali to Ubuntu Server
      + Open Firewall Dashboard by insert it IP Address on the Search space, like we have OPNsense "https://10.10.2.3/" , and login
      + Click the "OPNsense <" >> "Firewall" >> "Rules" >> "OPT1"
      + Action = Pass ; Interface = OPT1 ; Protocol = TCP ; Source = Single host or Network >> 10.10.2.114 ; Destination =  Single host or Network >> 10.10.3.198 ; Destination port range = from 22 to 22 ; Log = Enable ; Save 
      + Then click "Apply changes"

  * RDP from Kali to Windows 11
      + Open Firewall Dashboard by insert it IP Address on the Search space, like we have OPNsense "https://10.10.2.3/" , and login
      + Click the "OPNsense <" >> "Firewall" >> "Rules" >> "OPT1"
      + Action = Pass ; Interface = OPT1 ; Protocol = RDP ; Source = Single host or Network >> 10.10.2.114 ; Destination =  Single host or Network >> 10.10.3.199 ; Log = Enable ; Save
      + Then click "Apply changes"
  
  * review Obj1D3.png

    
* ### Remote Connection :

  * SSH from Kali to Ubuntu Server
      * Open Terminal and Tap "ssh kali@10.10.3.198" && Enter the Password 
      * Back to "OPNsense" >> "Firewall" >> "Log Files" >> "Live view"
      * Filter "Dst-port" "is" "22" && Click apply

  * RDP from Kali to Windows 11
      * Open Terminal and Tap "xfreerdp /v:10.10.3.199 /u:vboxuser2 /p:1234" && Click "y"
      * Back to "OPNsense" >> "Firewall" >> "Log Files" >> "Live view"
      * Filter "Dst-port" "is" "3389" && Click apply

  * review Obj2.1D3.png
  * review Obj2.2D3.png


* ### Review Logs :

  * #### 1) Ubuntu Server :
    + From the SSH connection Tap "sudo su" and enter password to enable Privileges 
    + Run ## grep "Accepted password" /var/log/auth.log
  
  * #### 2) Windows 11 :
    + From RDP connection Open Powershell as Administrator
    + Run ## Get-EventLog Security -InstanceId 4625 , 4624 >> "C:\\Users\\vboxuser2\\Desktop\\F&Slogs.txt"
  
  * review Obj3D3.png

  * #### 3) Deploy Wazuh agent :
    +  Search with browser for "https://10.10.3.177/" and enter credentials to login
    +  Click "Add agent" ; Select Windows MSI ; Set Name "wiin" ; and copy the cmd below ##
        +                 Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.3-1.msi -OutFile
                          Senv:tmp\wazuh-agent; msiexec.exe /i Senv:tmp\wazuh-agent /q WAZUH_MANAGER="10.10.3.177'
                          WAZUH_AGENT_NAME='wiin' 
        +                 Net Start Wazuh     
        ##
    +  Open Powershell as Administrator and past the cmd there ; comeback to Wazuh dashboard and the New agent will appear 
  * review Obj3.3.1.png
  * review Obj3.3.2.png


* ### Python :

  * #### 1) Write a function to count failed logins per IP from "auth.log" :
    + From SSH connection Tap "touch CountFailedLoginPerIP.py" then "nano CountFailedLoginPerIP.py"
      + Write in this file 
    + ##
      +       import re
              from collections import defaultdict

              def count_login_attempts(logpath='/var/log/auth.log'):
              failed = defaultdict(int)
              ip_pattern = r"(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})"
              
                  with open(logpath, 'r') as f:
                      for line in f:
                          if 'Failed password' in line:
                              match = re.search(ip_pattern, line)
                              if match:
                                  ip = match.group(0)
                                  failed[ip] += 1
                  all_ips = set(failed.keys()) 
                  for ip in sorted(all_ips):
                      print(f"{ip} failed: {failed[ip]}")
              count_login_attempts()

      ##
      + Then run python3 CountFailedLoginPerIP.py
  
  * review Obj4.1D3.pnj


  * #### 2) Write a function to count failed logins per IP from Exported Security Logs CSV :
    
  + Back to Powershell and run ##Get-WinEvent -FilterHashtable @{LogName="Security" ; ID= 4625} | Export-Csv -Path "C:\\Users\\vboxuser2\\Desktop\\newseclog.csv"##

    + Open new file "hello.py" and Write 
        + ##
          +       import re
                  from collections import defaultdict
  
                  def count_login_attempts(logpath='C:\\Users\\vboxuser2\\Desktop\\newseclog.csv'):
                  failed = defaultdict(int)
                  ip_pattern = r"(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})"
              
                      with open(logpath, 'r') as f:
                          for line in f:
                              if 'Adresse du' in line:
                                  match = re.search(ip_pattern, line)
                                  if match:
                                      ip = match.group(0)
                                      failed[ip] += 1
                      all_ips = set(failed.keys()) 
                      for ip in sorted(all_ips):
                          print(f"{ip} failed: {failed[ip]}")
                  count_login_attempts()
  
          ##
    + Run on Powershell "python3 hello.py"
  
  * review Obj4.2D3.pnj

### 5) Report Report: Document the SSH and RDP allow rule and log evidence:

* review ReportRDPfwRules.md
* review ReportSSHfwRules.md

* ## Observations / Findings :

     - Exporting and collecting log files and work with them from whatever you prefer can be easier than changing hosts every time .
     - Writing Rules Reports is a mandatory step to ensure precision of actions taken within particular process or service .


* ## Issues Encountered \& Solutions :

* #### Issue 1: VS-Code Extension did not apply .

   + Solution:
      1) Install Python and run it from Powershell as I've done With Linux Terminal  .
      2) Upload the log file "newseclog.csv" to Ubuntu server and work with it from it Terminal or with Kali Linux Desktop to simulate Daily Acts as SOC Analyst .


  * #### Issue 2 : Windows 11 doesn't support Filezilla by default .

    + Solution:
      1) Use SCP (Secure Copy Protocol), Tap ##scp newseclog.csv kali@10.10.3.198:/home/kali ##
      2) Use SFTP (Secure File Transport Protocol), Tap ## sftp kali@10.10.3.198 ## put C:\\Users\\vboxuser2\\Desktop\\newseclog.csv ##


* ## Conclusion / Next Step :
    * Start monitoring access control rules with OPNsense firewall
    * Create Python scripts this automates counting and analysis .
    * Successful deploy Wazuh agent .
    * Scanning different log sessions like RDP and SSH connections.
    * Next : 
        +  Write Rules Reports
        +  Enable Web Proxy Filters "Allow && Block" 
        +  Deploy Apache service on Ubuntu server.


* ## References :

. VS-Code: https://code.visualstudio.com

· OPNsense rule: https://docs.opnsense.org/manual/firewall.html

· Wazuh Windows agent: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-windows.html

· Event ID 4625: https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4624

. Windows 11 Pro: https://www.microsoft.com/en-us/software-download/windows11