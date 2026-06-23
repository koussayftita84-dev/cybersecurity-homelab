#   	Report

* ## Objective:
* Lab: Verify connectivity – ping from Kali to Windows 11 \&\& ping from kali to Ubuntu (INNET\_PROD) via OPNsense
* OPNsense: Check firewall live log – see ping traffic
* Python: Print Hello SOC, write script to read a copy of Security.evtx (exported)
* Python: Print Hello SOC, write script to read /var/log/syslog line by line
* Report: Draw your lab network topology (INNET\_SEC / INNET\_PROD)





* ## Lab Setup/Topology:
* INNET\_SEC:

      + Kali Linux --ipv4 (10.10.2.114)
    
      + Wazuh server --ipv4 (10.10.2.173)

* INNET\_PROD:

      + Windows 11 --ipv4 (10.10.3.101)
    
      + Ubuntu server --ipv4 (10.10.3.198)

* OPNsense:

      + LAN "em1" --ipv4 (10.10.2.3/24)
    
      + OPT1 "em2" --ipv4 (10.10.3.3/24)
    
      + WAN "em0" --ipv4 (DHCP)



* ## Step Performed:
  * ### Verify Connectivity \&\& Check firewall live log :

* #### 1) From Kali to Ubuntu server :

  - open Terminal and run cmd "ping 10.10.3.198"

  - open browser and search for "http://10.10.2.3"

  - login for OPNsense account "root" password "opnsense"

  - go to >>Firewall >>logfile >>live_view and write "icmp" on filter

* #### 2) From Kali to Windows :

	- back to Terminal and click "cntr+c" the run ping "10.10.3.101"

	- back again to OPNsense on >>Firewall >>logfile >>live_view

* ### Windows 11 Logs review with Python :

* #### 1) Export Security logs :

	- click "Win" and run Powershell as administrator\*

	- Tap ##Get-WinEvent -FilterHashtable @{LogName="Security" ; ID= 4625, 4624} | Select-Object Time-Created , Message | Export-Csv -Path "C:\\Users\\vboxuser\\Desktop\\newseclog.csv"##

  * #### 2) Python script :

    + open new file on VS-Code

      + Tap :
    ##

              import re

              print("hello soc")

              pattern= re.compile(r"ID du processus")

              pattern2= re.compile(r"Nom du processus")

              print(f"Select sec-log PID\\nN°1 PID Successful and Failed attemps\\nNo2Name Successful and Failed attemps ")

              current\_pattern=int(input())

              if current\_pattern == 1 :

                  current\_pattern=pattern

              elif current\_pattern == 2:

                  current\_pattern=pattern2

              with open ( "C:\\Users\\vboxuser\\Desktop\\newseclog. csv", "r", encoding= "utf-8") as infile:

                  for line\_num , line in enumerate(infile, start=1) :

                      match = current\_pattern.search(line)

                      if match:

                          match\_text = match. group()

                          print(f"line n°{line\_num} -- >{match\_text} {line.strip()}")
    ##
  

* ### Ubuntu server Logs review with Python :

  * #### 1) Remote connection :

  	- open terminal in kali and tap "sudo su" 

  	- run "ssh kali@10.10.3.198" and insert password 

  	- tap "sudo su" and "touch testlogs1.py" then nano "testlogs1.py"

  * #### 2) Python script :

  	+ tap:
    ##

            import re
    
            a=str("Hello SOC!\\nSelect the structure\\nN°1 (IP ADDRESS)\\nN°2 (SSH)\\nN°3 (PORT NUMBER)\\nN°4 (ERROR)")
    
            print(a)
    
            pattern1= re.compile(r"\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}")
    
            pattern2= re.compile(r"ssh")
    
            pattern3= re.compile(r"port \\d{1,9}\\")
    
            pattern4 = re.compile(r"ERROR")
    
            PN=int(input())
    
            with open("/var/log/auth. log", "r" , encoding= "utf-8") as infile, open("/home/kali/resultlogs1.txt", "w" , encoding= "utf-8") as outfile :
    
    
    
                if PN == 1:
        
                    for line\_num, line in enumerate(infile, start=1):
        
                        match = pattern1.search(line)
        
                        if match:
        
                            print(f"line °{line\_num}, {line.strip()}")
        
                            outfile.write(f"({match.group()} -> {line})")
        
        
        
                if PN == 2:
        
                    for line\_num, line in enumerate(infile, start=1):
        
                        match = pattern2. search(line)
        
                        if match:
        
                            print(f"line °{line\_num}, {line.strip()}")
        
                            outfile.write(f"({match.group()} -> {line})")
        
        
        
                if PN == 3:
        
                    for line\_num, line in enumerate(infile, start=1):
        
                        match = pattern3.search(line)
        
                        if match:
        
                            print(f"line °{line\_num}, {line.strip()}")
        
                            outfile.write(f"({match.group()} -> {line})")
        
        
        
                if PN == 4:
        
                    for line\_num, line in enumerate(infile, start=1):
        
                        match = pattern4.search(line)
        
                        if match:
        
                            print(f"line °{line\_num}, {line.strip()}")
        
                            outfile.write(f"({match.group()}-> {line})")
    ##



* ## Lab Network Topology :

  * review LabTopo.png



* ## Observations / Findings :
  * "UTF-8" for Windows logs exported as file.csv and Kali Linux /auth.log ; /Syslog .
  * BTW if we use ##Get-EventLog -LogName Security -InstanceId 4625 >> "C:\\Users\\vboxuser\\Desktop\\newseclog1.txt" ## ,We need to change it to "UTF-16"



* ## Analysis / Python Code :
  * As what we found in th observation session, we can change the method for Windows 11 for :

    + run in PowerShell  ##Get-EventLog -LogName Security -InstanceId 4624 >> "C:\\Users\\vboxuser\\Desktop\\newseclog1.txt" ##

    - and run ##Get-EventLog -LogName Security -InstanceId 4625 >> "C:\\Users\\vboxuser\\Desktop\\newseclog1.txt" ##

    + tap in VS-Code :
    ##

          import re

          print("hello soc")

          pattern= re.compile(r"4624")

          pattern2= re.compile(r"4625")

          print(f"Select sec-log PID\\nN°1 Successful attemps\\nN°2 Failed attemps")

          current\_pattern=int(input())

          if current\_pattern == 1 :

              current\_pattern=pattern

          elif current\_pattern == 2:

              current\_pattern=pattern2

          with open ( "C:\\Users\\vboxuser\\Desktop\\newseclog1", "r", encoding= "utf-16") as infile:

              for line\_num , line in enumerate(infile, start=1) :

                  match = current\_pattern.search(line)

                  if match:

                      match\_text = match.group()

                      print(f"line n°{line\_num} -- >{match\_text} {line.strip()}")
    ##


* ## Issues Encountered \& Solutions :
  * #### Issue 1: Windows 11 didn't support Python by default.

      + Solution: Install VS-Code and install Python Extension Inside it.

  * #### Issue 2: CSV export from wevtutil used UTF‑16 encoding. Python failed to read.

      +   Solution: Added encoding='utf-16' in open().



* ## Conclusion / Next Step :
  * Successfully start monitoring and analysis process in Lab environment.
  * Try some exploits and capture logs.



* ## References :

  + VS-Code: https://code.visualstudio.com

  + OPNsense rule: https://docs.opnsense.org/manual/firewall.html

  + Wazuh Windows agent: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-windows.html

  +  Event ID 4624: https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4624



