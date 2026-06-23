#     	Report

* ## Objective:

#### 1) OPNsense: Enable web proxy filter, block social media (test with Kali browser)

#### 2) Kali: Use curl to test blocked vs allowed sites

#### 3) Ubuntu: Install Apache, serve a test page

#### 4) Windows 11: Use curl (or browser dev tools) to test blocked vs allowed sites

#### 5) Python: 

   * Use dictionary to count occurrences of HTTP status codes in access.log
   * Python: Write script to parse IIS logs (if IIS installed) or simulate web log

#### 6) Wazuh (check /var/ossec/logs/archives) :

   * Verify Apache logs are being collected 
   * Verify Windows Event logs being collected


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


* ### 1) OPNsense: Enable web proxy filter && Block social media (test with Kali browser)

    * #### a) Enable  Web Proxy filter :
        + Open Firewall Dashboard by insert it IP Address on the Search space, like we have OPNsense "https://10.10.2.3/" , and login
        + Click the "OPNsense <" >> "Services" >> "Unbound DNS" >> "Blocklists" >> "+" >> Enable "Advanced mode" 
        + Select Witch type of DNS Blocklists you wish, like "Big blocklists (inc Ads)" ; "Pop-up Ads" ; "Gambling" ; "Social Networks"
        + Enable "Return NXDOMAIN" (it's critical to show you where the block comes from the DNS-BL) >> save >> Apply >> Enable the service
      * To ensure that the service is work you can search for Social Networks like "Facebook.com" ; "Reddit.com" 
      * Or use "nslookup [DNS or Ip_Address]" on Terminal, this allows you to solve if the service block the DNS request by printing "NXDOMAIN"

    * review Obj1D4.png

* ### 2) Kali: Use curl to test blocked vs allowed sites :

  + #### curl Commands to Test Allowed and Blocked Sites

| # | Purpose                                 | Command                                                                         | Notes                                                         |
|---|-----------------------------------------|---------------------------------------------------------------------------------|---------------------------------------------------------------|
| 1 | Basic verbose test                      | `curl -v https://www.google.com`                                                | Shows full request/response headers and connection details    |
| 2 | Check HTTP status code                  | `curl -o /dev/null -s -w "%{http_code}\n" https://www.google.com`               | Returns `200` (allowed), `403` (blocked), `000` (no response) |
| 3 | Follow redirects                        | `curl -L -v http://example.com`                                                 | Tracks redirects (e.g., to a block page)                      |
| 4 | Set timeout for blocked connections     | `curl --connect-timeout 5 -v https://blocked-site.example`                      | Prevents hanging when a firewall drops packets                |
| 5 | Test via a proxy                        | `curl -v -x http://proxy.example:8080 https://www.google.com`                   | Use if your network requires or enforces a proxy              |
| 6 | Change User-Agent                       | `curl -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64)" -v https://www.google.com` | Bypass simple user‑agent filters                              |
| 7 | Ignore SSL certificate errors           | `curl -k https://internal-site.example`                                         | For testing internal sites with self‑signed certificates      |
| 8 | Save response body and headers to files | `curl -D headers.txt -o body.html https://blocked-site.example`                 | Inspect block pages locally                                   |

 + Tap "curl -IsS [Uri or Ip_Address]" on Terminal

 * review Obj2D4.png

* ### 3) Ubuntu: Install Apache, serve a test page :

     * SSH from Kali to Ubuntu Server
        + Open Terminal and Tap "ssh kali@10.10.3.198" && Enter the Password
     * Run "sudo apt install apache2" and then run "sudo systemctl start apache2"
     * Back to browser on Kali and search for Ubuntu Server IP_Address 
  
 * review Obj3D4.png


* ### 4) Windows 11: Use curl (or browser dev tools) :

    * Open Win 11 Powershell as Administrator and run "curl -v https://www.facebook.com"
    * Then "curl -v https://www.google.com"

  * review Obj4D4.png


* ### 5) Python:

  * #### Use dictionary to count occurrences of HTTP status codes in access.log
    
    * Create new file.py name it "http_counter.py" in Ubuntu server and write : 
    ##
              import re
              from collections import defaultdict
              a=str("hello Soc")
              print(a)
              def count_http_attempts(logpath='/var/log/apache2/access.log'):
                failed_http = defaultdict(int)
                successfull_http = defaultdict(int)
                ip_pattern = r"(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})"
            
              with open(logpath, 'r') as f:
                for line in f:
                  if ' 404 ' in line:
                    match = re.search(ip_pattern, line)
                    if match:
                      ip= match.group(ø)
                      failed_http[ip] +=1
                  elif ' 200 ' in line:
                    match = re.search(ip_pattern, line)
                    if match:
                      ip= match.group(ø)
                      successfull_http[ip] +=1
            
              
              all_ips =set(failed_http.keys()) | set(successfull_http.keys())
              for ip in sorted(all_ips):
                print(f"{ip} failed 404 : {failed_http[ip]} succeed 200 : {successfull_http[ip]}")
              count_http_attempts()
    ##
    * Run it from Terminal "python3 http_counter.py"
    
   * review Obj5.1D4.png


  * #### Python: Write script to parse IIS logs (if IIS installed) or simulate web log

  * ##### 1) Generate fake logs :
    + Create a file.py name it "generatefakeiis.py" and write :
      ##   
             import random
             import re
            
             def generate_random_ip():
             return f"192.168.1.{random.randint(15,20)}"
            
             def random_date(ir):
             return f"2026-06-1{ir}"
            
            
             def S_port(a):
             b = a//2
             if b == 0 :
             return "Port:80, HTTP"
             elif b == 1 :
             return "Port:443, HTTPS"
            
            
             def SC_status(a):
             b =a//2
             if  b == 1:
             return "200 Successfull"
             else :
             return "404 Not found"
            
            
             def main():
            
                 with open('C:\\Users\\kouss\\Desktop\\eventlogs\\Fakelogsiis.txt','w',encoding='utf-8') as outfile:
                     MM = 22 ; SS = 10 ; S_ip = "10.10.3.198" ; CS = "Win-11 Microsoft-Edge" ; ir = 4
                     for i in range(2):
                        ir +=1
                        for _ in range(15):
                         MM +=1 ; SS +=1
                         time = f"17:{MM}:{SS}"
                         ip_address = generate_random_ip()
                         Date = random_date(ir)
                         a = random.randint(1,2)
                         Src_p= S_port(a)
                         Status = SC_status(a)
                         output=(f"\n#fields: {Date} {time} {ip_address} {CS} {S_ip} {Src_p} {Status}")
                         print(output)
                         outfile.write(output)
            
            
            
             if __name__ == "__main__":
             main()
  
      ##
    + Run it and watch logs from 'C:\\Users\\kouss\\Desktop\\eventlogs\\Fakelogsiis.txt'

   * review Obj5.21.png
    
  * ##### 2) Parse IIS logs :
    + Create a file.py name it "parse.py" and write :
        ##
              import re
              from collections import defaultdict
              a=str("hello Soc")
              print(a)
              def count_login_attempts(logpath='C:/Users/vboxuser2/Desktop/fakelogsiis.txt'):
                failed = defaultdict(int)
                Success = defaultdict(int)
                ip_pattern = r"(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})"
        
              with open(logpath, 'r') as f:
                for line in f:
                  if ' 404'in line:
                    match = re.search(ip_pattern, line)
                    if match:
                      ip= match.group(0)
                      failed[ip] +=1
                  if ' 200 ' in line:
                    match = re. search(ip_pattern, line)
                    if match:
                      ip= match.group(0)
                      Success[ip] +=1
        
              all_ips =set(failed.keys()) | set(Success.keys())
              for ip in sorted(all_ips):
                print(f"{ip} failed : {failed[ip]} | Success : {Success[ip]}")
              count_login_attempts()
        ##
    + Run it and watch logs from Terminal
  * review Obj5.22.png


* ### 6) Wazuh (check /var/ossec/logs/archives) :

* Verify Apache logs are being collected
   + SSH from Kali to Wazuh Server and redirect to archives by tapping "cd /var/ossec/logs/archives/2026/Jun"
   + Tap "ls -l" to display Logs collected 

* review Obj6.1D4.png


* Verify Windows Event logs being collected
   + We can find Win 11 Event logs from the same directory "cd /var/ossec/logs/archives/2026/Jun"
   + Or from Wazuh dashboard : Browzer >> Search for "10.10.3.177" and login >> Discover >> Filter agent.name "Wiin"

* review Obj6.2D4.png

* ## Observations && Findings :

    - Blocklist didn't apply until refreshing the service in Wazuh settings .
    - Browser also didn't apply the changes, so you need to oppen new window .



* ## Conclusion && Next Step :

  * Starting with firewall "OPNsense" by blocking unwanted websites like 'social networking' and 'gambling'
  * And test blocked URL with curl from "Kali" and "Win 11"
  * Then install Apache server in Ubuntu Server
  * Finally create python script to review logs from kali and parse iis logs from  Win 11
  * Next steps :
    + Simulate bruteforce attack to SSH && RDP connections 
    + Review logs from different security points 


* ## References :

. VS-Code: https://code.visualstudio.com

· OPNsense rule: https://docs.opnsense.org/manual/firewall.html

· Wazuh Windows agent: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-windows.html

· Event ID 4625: https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4624

. Windows 11 Pro: https://www.microsoft.com/en-us/software-download/windows11

. Ubuntu Server: https://ubuntu.com

. Apache Server: https://httpd.apache.org