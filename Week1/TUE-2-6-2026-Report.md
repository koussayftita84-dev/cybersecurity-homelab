#     	Report

* ## Objective:
* 1-
  + a) Kali: Use nmap to scan Windows 11 && Ubuntu server (basic: nmap -sS )
  + b) OPNsense: Capture scan in firewall logs, identify blocked ports
* 2-Windows 11: Check Security log for Event 4625 (failed logins)
* 3-Ubuntu: Check auth.log for failed SSH
* 4-Python: Write regex to extract IP addresses from a text log file
* 5-Enable RDP from Win 11 && test login from Kali

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
* ### kali: Use nmap to scan Windows 11  \&\& Check firewall live log :
* #### 1) From Kali to Windows 11 (TCP SYN stealth scan):

    	- open Terminal and run "nmap -sS 10.10.3.101"

    	- open browser and search for "http://10.10.2.3"

    	- login for OPNsense account "root" password "opnsense"

    	- go to >>Firewall >>logfile >>live\_view" select "src"/"is" and write "10.10.2.114" on filter

     >> Session blocked!
  * review Obj1.1D2.png

* #### 2)  From Kali to Ubuntu server:

    	- back to Terminal and run "nmap -sS 10.10.3.198"

    	- back to OPNsense on >>Firewall >>logfile >>live\_view" select "src"/"is" and write "10.10.2.114" on filter

     >> Only SSH port 22 is up
  * review Obj1.2D2.png


* ### Windows 11: Check Security log for Event 4625 (failed logins)
	- open Terminal and run##
              Get-EventLog -LogName Security -InstanceId 4625 -Newest 10 >> "C:\Users\vboxuser\Desktop\FLog25.txt"

  * review Obj1.2D2.png


* ### Ubuntu: Check auth.log for failed SSH
  * ####  Remote connection :

      - open terminal in Windows 11 and run "ssh kali@10.10.3.198"

      - run "sudo su" to enable privileges and insert password

      - tap ##
                grep "Failed password" /var/log/auth.log

    * review Obj3.1D2.png
    * review Obj3.2D2.png


* ### Python: Write regex to extract IP addresses from a text log file

  * Generate fake logs

    - Create a new file "Fakelogs.py"
      - Tap 
    ##
               
                 import random
                 import re
                 def generate_random_ip():
                     return f"192.168.1.{random.randint(15,20)}"
    
                 def check_firewall_rules(ip, rules):
                     for rule_ip, action in rules.items():
                         if ip == rule_ip:
                             return action
                     return "allow"
          
                 def main():
                     firewall_rules = {
                             f"192.168.1.{random.randint(0,20)}": "block",
                             f"192.168.1.{random.randint(0,20)}": "block",
                             f"192.168.1.{random.randint(0,20)}": "block",
                             f"192.168.1.{random.randint(0,20)}": "block",
                             f"192.168.1.{random.randint(0,20)}": "block",
                             f"192.168.1.{random.randint(0,20)}": "block"
                         }
          
                     print(firewall_rules)
                 with open('C:\\Users\\vboxuser\\Desktop\\Fakelogs.txt','w',encoding='utf-8') as outfile:
                         for _ in range(30):
                          ip_address = generate_random_ip()
                          action = check_firewall_rules(ip_address, firewall_rules)
                          random_number = random.randint(0,100)
                          if action=='block':
                             pid="4625"
                          else :
                             pid="4624"
                          outfile.write(f"ip addr {ip_address} : {action} -> {pid}\n")
                          a=(f"IP: {ip_address}, Action: {action}, Random: {random_number}")
                          print(a)
          
          
                 if __name__ =="__main__":
                  main()
      ##
* ### Write Python script to extract ip addresses

    + Tap in new file "ip_export.py":
        ##
    
                import re
    
                a=str("Hello SOC!\\nSelect the structure\\nN°1 (IP ADDRESS)\\nN°2 (SSH)")
    
                print(a)
    
                pattern1= re.compile(r"\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}")
    
                pattern2= re.compile(r"ssh")
    
                PN=int(input())
    
                with open('C:\\Users\\vboxuser\\Desktop\\Fakelogs.txt', "r" , encoding= "utf-8") as infile, open('C:\\Users\\vboxuser\\Desktop\\FakeIps.txt', "w" , encoding= "utf-8") as outfile :
    
    
    
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
        ##

  * review Obj4.1D2.png
  * review Obj4.2D2.png
  * review Obj4.3D2.png

* ### Enable RDP from Win 11 && test login from Kali

  * #### 1) Enable RDP :
    * Open Settings >> Search for Remote desktop >> Enable RDP 
    * Ensure that the firewall rule of allowing RDP is active
  * #### 2) Remote connection from Kali to Windows 11
    * Open Terminal and run "xfreerdp /v:[target_ip] /u:[target_user_name] /p:[target_password]"
    * For this situation i runs "xfreerdp /v:10.10.3.199 /u:vboxuser2 /p:1234"
    * review Obj5.2D2.png
  

* ## Observations / Findings :
     - Reviewing logs files without privilege escalation is by default denied


* ## Issues Encountered \& Solutions :
  * #### Issue 1: Windows 11 Family Edition doesn't support RDP .

    + Solution: 
      1) Install Windows 11 Multi-Edition ISO VM .
      2) Ensure that when you configure the new VM you have selected the Windows 11 Pro/x64 or Windows 11 Education/x64
  * #### Issue 2 : After successful RDP connect from Kali to Win 11, the session will log out if you open Win 11 VM again .
    + Solution:
      +  Stay at the window that appears in Kali VM .

* ## Conclusion / Next Step :
  * Successful scanning and analysis process in Lab environment.
  * Implement and then open RDP connection.
  * Next : Try some exploits and capture logs.



* ## References :

. VS-Code: https://code.visualstudio.com

· OPNsense rule: https://docs.opnsense.org/manual/firewall.html

· Wazuh Windows agent: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-windows.html

· Event ID 4625: https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4624

. Nmap : https://nmap.org