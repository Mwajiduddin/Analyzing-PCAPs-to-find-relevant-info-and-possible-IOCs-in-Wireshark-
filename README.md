<h1 align="center">Analyzing PCAPs to find relevant info and possible indicators of compromise (IOCs) in Wireshark</h1>

In this lab from SBT’s Blue Team Level 1 course we’ll be analyzing three different PCAPs to find pertinent data that shows evidence of a compromise.
<p align="center">
<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20163048.png" />
</p>

<h2>PCAP 1:</h2>
<h3 align="center">Open the first PCAP file and find if there’s any evidence of host discovery scanning going on in the network. What is the IP address performing the scan and what protocol is used?</h3> 
By opening this PCAP file, from the get go the initial packets show evidence of scanning for host discovery. The biggest tell is the Info column where 192.168.56.1 is broadcasting on the network asking every IP in the subnet what their MAC address is by using the ARP protocol. 
Another way to be certain of IP address performing the scan is by expanding the Address Resolution Protocol section at the bottom:


<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20164000.png" />
</p>

<h3 align="center">What IP address is being port scanned by the malicious IP?</h3> 
In the first question, we figured out that 192.168.56.1 is the IP that's scanning the network so it stands to reason that this is the malicious IP. Easiest way to see all the ports in a PCAP is Statistics > Conversations > TCP tab:

<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20164529.png" />
</p>
Here we can see that 192.168.56.1 is talking to 192.168.56.111 using these different ports and clear evidence of port scanning. 


<h3 align="center">Take a closer look at the FTP traffic in this PCAP. How many users are allowed to connect to the FTP server at once?</h3> 
To filter for just FTP traffic, just type ‘ftp’ in the display filter or Statistics > Protocol Hierarchy > Right-click on FTP and Apply as Filter. 

<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20165016.png" />
</p>
Looking at the Hex Dump & ASCII section at the bottom, we can see how many users can be connected to the FTP server at once which is 50:

<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20165340.png" />
</p>

<h3 align="center">The malicious attacker tried logging into the FTP server using “anonymous” as the username but what was the incorrect password?</h3> 
Just by looking at the packet list we can see the username and password (IEUser@) the malicious attacker tried to use to login.

<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20165824.png" />
</p>
Another way to check just to be sure is to right click on the packet that shows the wrong password and select Follow > TCP Stream and you’ll get the TCP stream window showing the communication between the FTP server and the malicious client:

<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20165938.png" />
</p>

<h3 align="center">Export the robots.txt 404 page from the packet #4612 as a HTTP Object and open the text file. What version of Apache is running?</h3> 
File > Export Objects > HTTP > select packet #4612 and Save > text file shows that it's Apache 2.4.38

<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20170450.png" />
</p>

<h2>PCAP 2:</h2>

<h3 align="center">Open PCAP 2 and figure out which IP address downloaded the ZIP file.</h3> 
There are 329 packets in this PCAP, you can individual go by each packet and find out which one downloaded the ZIP file but that’s time consuming, the easier way: File > Export Objects > HTTP > you’ll see that packet 323 mentions a ZIP file.

<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20171112.png" />
</p>
But don’t be fooled, just because you see an IP next to it doesn’t mean that’s the IP address that downloaded the file. If you read the packet header section for packet 323, you’ll find that 192.168.56.111 downloaded the file:
<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20171545.png" />

<h3 align="center">What is the source port and destination port for the downloaded ZIP file?</h3> 
Just read the packet header section:
<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20171718.png" />

<h3 align="center">What is the name of the ZIP file?</h3> 
File > Export Objects > HTTP > packet 323:
<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20171906.png" />

<h3 align="center">Export the ZIP file and find out the MD5 hash value for the ZIP file?</h3> 
File > Export Objects > HTTP > select packet 323 > Save > open Terminal > type md5sum cr4ckx0r.zip
<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20172319.png" />

<h2>PCAP 3:</h2>

<h3 align="center">Which IP address is running a FTP server?</h3> 
Either type ‘ftp’ in the display filter or Statistics > Protocol Hierarchy > Right-click on FTP and Apply as Filter.
Here we can see the initial packets note a response of 220 between source IP of 192.168.56.118 and destination IP of 192.168.56.111. A server sends a 220 response code to tell that it’s ready for ready for input command from a client so that means the source IP of 192.168.56.118 is the FTP server.
<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20172857.png" />

<h3 align="center">At what time did the attacker send the first password in a dictionary attack to the FTP server?</h3> 
If we scroll down the list of packets, we can see that see that the client (attacker) is sending different numerous but common passwords to the FTP server with the first one being 123456:
<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20173803.png" />
(Remember to change your time display: View > Time Display Format)

<h3 align="center">At what time did the attacker successfully log into the FTP server and using which password to do so?</h3> 
Starting off from the previous question, keep scrolling and you’ll see that the attacker is still trying to get through using different passwords until you reach to packet # 934, where it shows the attacker finally got the password right. But the packets above packet # 934 shows all these different passwords attempted by the attacker so which one is the correct password:
<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20174606.png" />

Right-click on packet #934 > Follow > TCP Stream > you’ll see that naruto was the correct password:
<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20174934.png" />

<h3 align="center">What was the name of the file the attacker downloaded from the FTP server?</h3> 
Continuing where we were left off from the previous question, keep scrolling and we’ll see that the attacker is requesting a sensitively named file from the FTP server:
<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20175406.png" />
Right-click  > Follow > TCP Stream and you’ll get a clearer picture that the attacker requested the file named password.backup from the FTP server. (RETR is an abbreviation for retrieve that tells the FTP server to send the specified file to the client)
<img src="https://github.com/Mwajiduddin/Analyzing-PCAPs-to-find-relevant-info-and-possible-IOCs-in-Wireshark-/blob/main/Screenshot%202026-06-08%20175529.png" />
