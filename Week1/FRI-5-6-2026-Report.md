#     	Report

* ## Objective:

#### 1) Kali: 
  * Simulate a brute force attack against Ubuntu SSH (hydra)
  * Ubuntu: Watch auth.log in real time (tail -f)
  * Simulate a brute force RDP attack against Windows 11 (hydra or crowbar)
  * Windows 11: Use Event Viewer to watch Security log (filter 4624, 4625) in real time

#### 2) Wazuh: 
  * Check for brute force alert (SSH brute force rule)
  * Check for brute force alert (RDP brute force rule)

#### 3) Python:
  * Parse auth.log, output JSON report of attacking IPs
  * Parse exported Security log CSV, output JSON report of attacking IPs

#### 4) Report: Write a 1-page incident summary with containment suggestion


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


* ### 1) Kali:
  * #### Simulate a brute force attack against Ubuntu SSH (hydra) :
    * Open Terminal in Kali Linux 
    * Write :
    ##
              $# sudo su
              $# hydra -L users.txt -P passtest.txt 10.10.3.198 ssh -t1 -o result10.txt 
    ##
  * #### Ubuntu: Watch auth.log in real time (tail -f)
    * Open another Terminal CLI 
    * Connect with SSH to Ubuntu Server by Taping "ssh kali@10.10.3.198" && Enter password
    * Run "sudo su" && "tail -f /var/log/auth.log" to view Logs in real time

 * review Obj1.1D5.png

   * #### Simulate a brute force RDP attack against Windows 11 (hydra or crowbar)
     * From Kali Terminal write :
     ##
             $# hydra -L users.txt -P passtest.txt 10.10.3.199 rdp -t1 -o result10.txt 
     ##
   * #### Windows 11: Use Event Viewer to watch Security log (filter 4624, 4625) in real time
  
 * review Obj1.2D55.png

* ### 2)  Wazuh:
  * #### Check for brute force alert (SSH brute force rule)
    * Open Browser and search for "https://10.10.2.173/"
    * Enter Username: admin ; Password: admin to login
    * Go to "Discover" >> Filter for "rule.description: syslog: User missed the password more than one time"

 * review Obj2.1D5.png

   * #### Check for brute force alert (RDP brute force rule)
     * Stay on "Discover" >> Filter for "agent.name: wiin"
   
 * review Obj2.2D5

* ### 3) Python:

* #### Parse auth.log, output JSON report of attacking IPs :
  * Open another Terminal CLI
  * Connect with SSH to Ubuntu Server by Taping "ssh kali@10.10.3.198" && Enter password
  * Create a new file "Attack_Report.py" && Write:
  ##

           import json
           import re
           import random
           from collections import defaultdict
            
           logpath='/var/log/auth.log'
           outfile='/home/kali/FakeAlerts.json'
           def Action(failed, success):
              if (failed > 7) and (success == 0) :
                  return "Bruteforce Attempts"
              elif (failed > 4) and (success > 1) :
                  return "Server is Compromized"
            
           def Id():
              return random. randint(1000, 9999)
            
           def count_login_attempts():
              failed = defaultdict(int)
              success = defaultdict(int)
              ip_pattern =r"(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})"
            
               with open(logpath, 'r', encoding='utf-8') as f:
                   with open(outfile, 'w', encoding="utf-8") as Out:
                       for line in f:
                           if 'Failed password' in line:
                               match = re.search(ip_pattern, line)
                               if match:
                                   ip= match.group(0)
                                   failed[ip] +=1
                           elif 'Accepted password' in line:
                               match = re. search(ip_pattern, line)
                               if match:
                                   ip= match.group(0)
                                   success[ip] +=1
                       all_ips =set(failed.keys()) | set(success.keys())
                       for ip in sorted(all_ips):
                           a=Id()
                           print(f"{a} {Action(failed[ip], success[ip])} {ip :< 15} :\n  * failed :{failed[ip]:<5} \n * succeed :{success [ip]} ")
                           logs={
                           "Alert": f"{Action(failed[ip], success[ip])}",
                               "_id": f"{a}",
                               "_Src-IP": f"{ip :< 15}",
                               " failed": f"{failed[ip] :< 5}",
                               "succeed": f"{success[ip]}",
                               "_Dst-IP": f"10.10.3.198"
                           },
                       json.dump(logs, Out, indent=4)
           count_login_attempts()
  ##
   * review Obj3.1D5.png

* #### Parse exported Security log CSV, output JSON report of attacking IPs :
  * Open Event-viewer >> Win journal >> Security | Filter for event "4624" , "4625"
  * Export logs as "syslogs11.csv"
  * Create new file "hello.py" && Write:
  ##
         import re
         import json
         import csv
         import random
         from collections import defaultdict
         a=str("hello Soc")
         print(a)
      
         def Action(failed, success):
           if (failed > 7) and (success == 0):
             return "Brutforce Attempts"
           elif (failed > 7) and (success >= 1):
             return "Server is Compromized"
        
         def Id():
            return random.randint(1000,9999)
        
         def count_login_attempts(logpath='C:/Users/vboxuser2/Desktop/syslogs11.csv'):
            success = defaultdict(int)
            failed = defaultdict(int)
            ip_pattern = r"(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})"
            with open(logpath, 'r', encoding='utf-8') as f :
                with open("C:/Users/vboxuser2/Desktop/Fakelogs.json", 'w' , encoding='utf-8') as Out:
                  reader = csv.reader(f, delimiter=',')
                  next(reader, None)
                  for row in reader:
                      if len(row) > 5:
                          message = row[5]
                          if ('ou mot de passe incorrect') in message and ('Adresse du reseau source') in message:
                              match = re.search(ip_pattern, message)
                              if match:
                                ip= match.group(0)
                                failed[ip] +=1
                          if "ID d'ouverture de session" in message and "Adresse du reseau source" in message:
                              match = re.search(ip_pattern, message)
                              if match:
                                ip= match.group(e)
                                success[ip] +=1
        
                  all_ips =set(failed.keys()) | set(success.keys())
                  for ip in sorted(all_ips):
                      a= Id()
                      print(f"ID:{a} | {Action(failed[ip], success[ip])}\n {ip}\n failed : {failed[ip]} \n success : {success[ ip] }")
                      logs={
                        "Alert": f"{Action(failed[ip], success[ip])}",
                        " id": f"{a}",
                        "Src-IP": f"{ip}",
                        "_failed": f"{failed[ip]}",
                        "succeed": f"{(success[ip])}",
                        " Dst-IP": "10.10.3.199",
                        "_Dst-Port": "3389",
                      },
                  json.dump(logs, Out, indent=4)
         count_login_attempts()
  ##
  * review Obj3.2D5.png

  

* ### 4) Report: Write a 1-page incident summary with containment suggestion:

  * review Incident-Response-Report-5-6-2026.md


* ## Issues Encountered & Solutions :

* #### Issue 1:Time Synchronizing between endpoints

  + Solution:
    + Set-up Timer Synchronizer like "Chronic" for Linux.
    + Add NTP Server to the Network.


* #### Issue 2: "ValueError: I/O operation on closed file"
  * This structure is invalid: 
     ## 
          with open(logpath, 'r', encoding='utf-8') as f , open("C:/Users/vboxuser2/Desktop/Fakelogs.json", 'w' , encoding='utf-8') as Out:
     ##
  + Solution:
    ##
         with open(logpath, 'r', encoding='utf-8') as f :
            with open("C:/Users/vboxuser2/Desktop/Fakelogs.json", 'w' , encoding='utf-8') as Out:
    ##


* ## Conclusion & Next Step :
+
  + Simulate Bruteforce Attacks && Review Alerts and Logs.
  + Write python script to parse logs and export it as file.json format.
_________________________________________________________________________________
+

  + Create Rules to detect failed RDP, SSH and sudo attempts.
  + Parse EVTX file format.


* ## References :

. VS-Code: https://code.visualstudio.com

· OPNsense rule: https://docs.opnsense.org/manual/firewall.html

· Wazuh Windows agent: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-windows.html

· Event ID 4625: https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4624

. Windows 11 Pro: https://www.microsoft.com/en-us/software-download/windows11
