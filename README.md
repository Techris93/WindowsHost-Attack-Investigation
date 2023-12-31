# WindowsHost-Attack-Investigation

### Objectives

- Part 1: Investigate the Attack with Sguil
- Part 2: Use Kibana to Investigate Alerts


### Scenario

During March 2019, network security monitoring tools detected a malware infection on a Windows computer within the network. The task is to examine the alerts and respond to the following inquiries:
- What was the specific time of the attack on 2019-03-19?
- Which Windows host computer was infected?
- Who was the user?
- What was the computer infected with?

### Required Resources

- Security Onion virtual machine
- Internet access

### Method

**Part 1: Investigate the Attack with Sguil**

Step 1: Open Sguil and locate the alerts on 3-19-2019.

a. Login to Security Onion VM with username and password.

b. Launch Sguil from the desktop. Login with username and password. Click Select All and Start Sguil to view all the alerts generated by the network sensors.

c. Locate the group of alerts from 19 March 2019.

According to Sguil, the timestamps for the first and last of the alerts that occurred on 3-19-2019 are:
- 01:45:03 to 04:54:34.
- The alerts show suspicious activity for a little over 3 hours but the majority of the alerts took place for a little over 3 minutes and 43 seconds.

Step 2: Review the alerts in detail.

a. In Sguil, click the first of the alerts on 3-19-2019 (Alert ID 5.439). Make sure to check the Show Packet Data and Show Rule checkboxes to examine the packet header information and the IDS signature rule related to the alert. Right on the Alert ID and pivot to Wireshark. 

![image](https://i.imgur.com/DAwfZGi.png)
Based on the information derived from this initial alert:
- the source IP address and port number and destination IP address and port number are: Source: 10.0.90.215:52609, Destination: 10.0.90.9:53
- the type of protocol and request/response involved are: UDP, Dynamic DNS, update and response
- the IDS alert and message are: Alert udp $EXTERNAL_NET any -> $HOME_NET 53, msg: “ET POLICY DNS Update from External net
- This alert may be the result of a misconfiguration in the IDS because the DNS request was a Dynamic DNS update from an internal host to a DNS server on the internal network and not from an external network to the internal network.
- the hostname, domain name, and IP address of the source host in the DNS update are: Bobby-Tiger-PC, littletigers.info, 10.0.90.215

b. In Sguil, select the second of the alerts on 3-19-2019. Right-click the Alert ID 5.440 and select Transcript.
![image](https://i.imgur.com/WWgfrhy.png)
![image](https://i.imgur.com/rpcncNb.png)
- From the transcript the source and destination IP address and port numbers are: Source 10.0.90.215:49204 and Destination 209.141.34.8:80
- Looking at the request (blue), the request is: GET /test1.exe
- The initial character of this file is MZ, a Windows executable .exe or .dll file

c. Close the transcript. Use Wireshark to export the executable file for malware analysis (File > Export Objects > HTTP…). Save the file to the analyst’s home folder.

d. Open a terminal in Security Onion VM and create a SHA256 hash from the exported file. Use the following command:
```
sha256sum test1.exe
```
![image](https://i.imgur.com/xidfTom.png)

e. Copy the file hash and submit it to the Cisco Talos file reputation center at https://talosintelligence.com/talos_file_reputation.

![image](https://i.imgur.com/M2gmrCL.png)
![image](https://i.imgur.com/DRvVQZM.png)

- Talos recognized the file hash and identified it as malware of kind win32 trojan-spy-agent.

f. In Sguil select the alert with Alert ID 5.480 and the Event Message Remcos RAT Checkin 23.
![image](https://i.imgur.com/8T4w8xu.png)

g. Right-click the Alert ID and select Transcript. Scroll through the transcript.
![image](https://i.imgur.com/EDIV5MC.png)
- The destination port is 2404 and it is not a well-known port.
- The communication is encrypted.
- Remcos RAT uses multiple packers, base64 encoding, and RC4 encryption to bypass detection and throw off security analysts

h. Using Sguil and the remaining alerts from 3-19-2019:

- The alert IDs alert to a second executable file being downloaded are 5.483, 5.485, 5.497, 5.509, 5.521, 5.533.
- The server IP address and port number the file was downloaded from is 217.23.14.81:80
- The name of the file that was downloaded is F4.exe

Create a SHA256 hash of the file and submit the hash online at the Cisco Talos File Reputation Center to see if it matches known malware.
- The executable file is a known malware and of the type PE32 executable
- The AMP DETECTION NAME is trojan downloader Win.Dropper.Cridex::1201

i. Examine the remaining three alerts from 3-19-2019 by looking at the header information in Show Packet Data, the IDS signature in Show Rule, and the Alert ID Transcripts.

- All three alerts are encrypted and all three alerts were triggered by a blacklisted malicious SSL certificate – Dridex

j. There may be additional related information available in Kibana. Close Sguil and launch Kibana from the desktop.

**Part 2: Use Kibana to Investigate Alerts**

- Use Kibana to further investigate the attack on 3-19-2019.

Step 1: Open Kibana and narrow the timeframe.

a. Login to Kibana with the username and password.

b. Open Kibana, click Last 24 Hours and the Absolute time range tab to change the time range from March 1, 2019, to March 31, 2019.

c. The Total Log Count Over Time timeline will show an event on March 19. Click that event to narrow the focus to the specific time range of the attack.

![image](https://i.imgur.com/HKbVR7j.png)

Step 2: Review the alerts in the narrowed timeframe.

a. In the Kibana dashboard scroll down to the All Sensors – Log Type visualization.

![Image](https://i.imgur.com/IzuIEEH.png)

b. Scroll down and notice that the NIDS Alert Summary in Kibana has many of the same IDS alerts as listed in Sguil. Click the magnifier to filter on the second alert ET TROJAN ABUSE.CH SSL Blacklist Malicious SSL certificate detected (Dridex) from Source IP Address 31.22.4.176.

![image](https://i.imgur.com/U1aNtBb.png)

c. Scroll down to All Logs and click the arrow to expand the first log in the list with the source IP address 31.22.4.176.

![image](https://i.imgur.com/dgOp8oz.png)
![image](https://i.imgur.com/wpHAu35.png)

- The country and city location for this alert is: United Kingdom, Newcastle upon Tyne

![image](https://i.imgur.com/twfyg7d.png)

What is the geo country and city for the alert from 115.112.43.81?
India, Mumbai

d. Scroll back to the top of the page and click the Home link under Navigation.

e. Earlier we noted log types like bro_http listed in the Home dashboard. You can filter for the various log types but the built-in dashboards will probably have more information. Scroll back to the top of the page and click HTTP in the dashboard link under Zeek Hunting in Navigation.

![image](https://i.imgur.com/I4qHsOO.png)
![image](https://i.imgur.com/e2mEtVM.png)
![image](https://i.imgur.com/XhskiZG.png)

- The Log Count in the HTTP dashboard is 4.
- From the United States and the Netherlands.
- The URIs for the files that were downloaded are: /f4.exe, /ncsi.txt, /pki/crl/products/CSPCA.crl, and /test1.exe

f. Scroll back to the top of the web page and under Navigation – Zeek Hunting click DNS. Scroll to the DNS Queries visualization.
![image](https://i.imgur.com/I8bClb7.png)
![image](https://i.imgur.com/9khAkPg.png)
![image](https://i.imgur.com/LQ6Dih5.png)

- Four(4) detection engines recognized the URLs as malicious.

### Further Investigations
- Match the HTTP – URIs to the HTTP – Sites on the dashboard.
- What are the CSPCA.crl and ncsi.txt files related to. Use a web browser and a search engine for additional information.
- examining the following Zeek Hunting dashboards:

a. DCE/RPC – for information about the Windows network remote procedures and resources involved

b. Kerberos – for information on the hostnames, and domain names that were used

c. PE – for information on the portable executables

d. SSL and x.509 – for information on the security certificate names and countries that were used

e. SMB – for more information on the SMB shares on the littletigers network

f. Weird – for protocol and service anomalies and malformed communications

Note: CSPCA.crl is a request for the Microsoft certificate revocation list and ncsi.txt refers to the network connection status indicator and is used automatically by Windows hosts as a self-test to verify online connectivity.

## References
- https://malware-traffic-analysis.net
- https://contenthub.netacad.com/cyberops
- https://superuser.com/questions/427967/what-would-http-crl-microsoft-com-pki-crl-products-cspca-crl-be-used-for
- https://campus.barracuda.com/product/websecuritygateway/knowledgebase/50160000000auwRAAQ/should-i-block-http-www-msftncsi-com-ncsi-txt-on-my-barracuda-product/
