                                                                                                                                             
┌──(kali㉿kali)-[~]
└─$ sudo nmap -p- -sS --min-rate 5000 -n -Pn 10.129.238.152
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-10 20:31 EDT
Stats: 0:00:17 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 66.76% done; ETC: 20:32 (0:00:08 remaining)
Nmap scan report for 10.129.238.152
Host is up (0.046s latency).
Not shown: 65514 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
2222/tcp  open  EtherNetIP-1
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
52699/tcp open  unknown
53123/tcp open  unknown
53142/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 26.44 seconds
                                                                                                                                             
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC -sV -p 53,88,135,139,389,445,464,539,636,2222,3268,3269,5985,9389,49664,49668,49670,49671,52699,53123,53142  10.129.238.152 

Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-10 20:35 EDT
Nmap scan report for dc.voleur.htb (10.129.238.152)
Host is up (0.096s latency).

PORT      STATE    SERVICE       VERSION
53/tcp    open     domain        Simple DNS Plus
88/tcp    open     kerberos-sec  Microsoft Windows Kerberos (server time: 2025-07-11 08:35:14Z)
135/tcp   open     msrpc         Microsoft Windows RPC
139/tcp   open     netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open     ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
445/tcp   open     microsoft-ds?
464/tcp   open     kpasswd5?
539/tcp   filtered apertus-ldp
636/tcp   open     tcpwrapped
2222/tcp  open     ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 42:40:39:30:d6:fc:44:95:37:e1:9b:88:0b:a2:d7:71 (RSA)
|   256 ae:d9:c2:b8:7d:65:6f:58:c8:f4:ae:4f:e4:e8:cd:94 (ECDSA)
|_  256 53:ad:6b:6c:ca:ae:1b:40:44:71:52:95:29:b1:bb:c1 (ED25519)
3268/tcp  open     ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
3269/tcp  open     tcpwrapped
5985/tcp  open     http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open     mc-nmf        .NET Message Framing
49664/tcp open     msrpc         Microsoft Windows RPC
49668/tcp open     msrpc         Microsoft Windows RPC
49670/tcp open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
49671/tcp open     msrpc         Microsoft Windows RPC
52699/tcp open     msrpc         Microsoft Windows RPC
53123/tcp open     msrpc         Microsoft Windows RPC
53142/tcp open     msrpc         Microsoft Windows RPC
Service Info: Host: DC; OSs: Windows, Linux; CPE: cpe:/o:microsoft:windows, cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2025-07-11T08:36:06
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 7h59m59s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 98.88 seconds
                                                                                                                                             
┌──(kali㉿kali)-[~]
└─$ 
