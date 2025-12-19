# FTP 
File transfer protocol is often used to share files within an org, it operaters on the application layer, similar to HTTP. Runs natively on port 21. TFTP is simpler than FTP (trivial) and it runs on UDP far more insicure than FTP.
```
Commands	Description
connect	Sets the remote host, and optionally the port, for file transfers.
get	Transfers a file or set of files from the remote host to the local host.
put	Transfers a file or set of files from the local host onto the remote host.
quit	Exits tftp.
status	Shows the current status of tftp, including the current transfer mode (ascii or binary), connection status, time-out value, and so on.
verbose	Turns verbose mode, which displays additional information during file transfer, on or off.
```

Default conf path to see what settings are availeble can be found at **/etc/vsftpd.conf**.

Default location where users are defined to be able to user FTP-
**/etc/ftpusers**
**Dangerous Settings**
```
anonymous_enable=YES	Allowing anonymous login?
anon_upload_enable=YES	Allowing anonymous to upload files?
anon_mkdir_write_enable=YES	Allowing anonymous to create new directories?
no_anon_password=YES	Do not ask anonymous for password?
anon_root=/home/username/ftp	Directory for anonymous.
write_enable=YES	Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE?
```
if anon login is allowed the following command can be used to dowload all files
```bash
 wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```
# SMB Shares
```bash
└─$ sudo nmap -sC -sV  10.129.202.5 
445/tcp  open  netbios-ssn Samba smbd 4
└─$ smbmap -H 10.129.202.5                   
        sambashare                                              READ ONLY       InFreight SMB v3.1
└─$ smbclient //10.129.202.5/sambashare -N
smb: \> ls
  .                                   D        0  Mon Nov  8 08:43:14 2021
  ..                                  D        0  Mon Nov  8 10:53:19 2021
  .profile                            H      807  Tue Feb 25 07:03:22 2020
  contents                            D        0  Mon Nov  8 08:43:45 2021
  .bash_logout                        H      220  Tue Feb 25 07:03:22 2020
  .bashrc                             H     3771  Tue Feb 25 07:03:22 2020

smb: \contents\> ls
  .                                   D        0  Mon Nov  8 08:43:45 2021
  ..                                  D        0  Mon Nov  8 08:43:14 2021
  flag.txt                            N       38  Mon Nov  8 08:43:45 2021
  smb: \contents\> get flag.txt
getting file \contents\flag.txt of size 38 as flag.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)

└─$ ./enum4linux-ng.py 10.129.202.5 -A 
└─$ rpcclient -U "" -N 10.129.202.5
netname: sambashare
        remark: InFreight SMB v3.1
        path:   C:\home\sambauser\
        password:
rpcclient $> exit
```

# nfs

``bash                                                                                                                     
┌──(kali㉿kali)-[~]
└─$ sudo nmap --script nfs* 10.129.202.5 -sV -p111,2049
```

```bash
─$ showmount -e 10.129.202.5                          
Export list for 10.129.202.5:
/var/nfs      10.0.0.0/8
/mnt/nfsshare 10.0.0.0/8
```

```bash 
┌──(kali㉿kali)-[~]
└─$ sudo mount -t nfs 10.129.202.5:/var/nfs ./target -o nolock                                                                               
┌──(kali㉿kali)-[~]
└─$ sudo mount -t nfs 10.129.202.5:/mnt/nfsshare ./target -o nolock                                                                                                                                            
┌──(kali㉿kali)-[~]
└─$ cd target                                                                                                                                   
┌──(kali㉿kali)-[~/target]
└─$ tree .
└── flag.txt
1 directory, 1 file
```


dnsrecon -d inlanefreight.htb -n 10.129.203.75 -D /home/kali/SecLists-master/Discovery/DNS/subdomains-top1million-20000.txt -t brt


└─$ dig @10.129.203.75 internal.inlanefreight.htb AXFR
└─$ dig axfr internal.inlanefreight.htb @10.129.203.75

└─$ gobuster dns -d inlanefreight.htb -r 10.129.203.75 -w /home/kali/SecLists-master/Discovery/DNS/subdomains-top1million-5000.txt 

amass enum -d inlanefreight.htb -brute -recursive -w dns-Jhaddix.txt


**SMTP**
The SMTP Footprinting module focuses on understanding how SMTP works, how it is commonly deployed, and how misconfigurations can leak valuable information during enumeration. SMTP is primarily used for sending email and typically runs on port 25, with newer authenticated submissions occurring on ports 587 or 465 using STARTTLS for encryption. By default, SMTP transmits data in plaintext, which makes it an attractive target during reconnaissance.

A key takeaway is that SMTP servers can unintentionally disclose valid system users through commands such as VRFY and EXPN. Depending on server configuration, response codes like 252 may indicate that a mailbox exists, while 550 confirms a user does not exist. However, the module emphasizes that enumeration results should never be blindly trusted, as some servers are configured to return misleading responses. Manual verification and contextual analysis are critical.

The module also highlights the danger of open relay configurations, which allow unauthenticated users to send email through the server. Misconfigured relay settings can enable spam campaigns, spoofed emails, and information leakage. Tools such as telnet, netcat, and Nmap SMTP NSE scripts are used to identify supported commands, authentication mechanisms, and relay behavior.

Overall, the module teaches how SMTP misconfigurations can expose internal usernames, employee accounts, and infrastructure details, making SMTP a valuable early foothold during penetration testing and internal network assessments.

**Key Commands to run on SMTP**
```
Command	Usage	Description
HELO	HELO <hostname>	Initiates the SMTP session and identifies the client to the server.
EHLO	EHLO <hostname>	Extended HELO; lists supported ESMTP features (AUTH, STARTTLS, SIZE, etc.).
MAIL FROM	MAIL FROM:<address>	Specifies the sender’s email address for the message.
RCPT TO	RCPT TO:<address>	Specifies the recipient’s email address; may be used for user enumeration.
DATA	DATA	Begins the email body; message ends with a single period (.) on its own line.
VRFY	VRFY <username>	Verifies whether a mailbox or local user exists on the server.
EXPN	EXPN <mailing-list>	Expands a mailing list into individual recipient addresses.
AUTH	AUTH LOGIN / AUTH PLAIN	Initiates SMTP authentication using encoded credentials.
STARTTLS	STARTTLS	Upgrades an unencrypted SMTP connection to TLS encryption.
RSET	RSET	Resets the current mail transaction without closing the connection.
NOOP	NOOP	Sends a keep-alive request to prevent session timeout.
QUIT	QUIT	Terminates the SMTP session gracefully.
HELP	HELP	Requests a list of supported SMTP commands (often disabled).
```
**Typical responces**
```
🔑 Key Response Codes (Quick Reference)
Code	Meaning
250	Requested action completed successfully
252	User exists but verification is restricted
354	Server ready to receive message data
550	Mailbox does not exist / access denied
503	Bad command sequence
502	Command not implemented\
```

**Banner Grab SMTP version**
```bash
nc -nv 10.10.10.10 25
```

**Enumerate port 25 SMTP**
```bash
sudo nmap 10.129.218.123 -p25 --script smtp-open-relay -v
```

**Connect to SMTP server via 25 and telnet can run commands to enumerate**
```bash
telnet 10.129.218.123 25
```
**While loop to enumerate usernames that are present on the host**
```bash
 while read user; do               
  printf "VRFY %s\r\nQUIT\r\n" "$user" | nc -nv 10.129.218.123 25 | \
  grep -E "550|250|252" | sed "s/^/$user -> /"
done < /home/kali/SecLists-master/Usernames/top-usernames-shortlist.txt
```

# HTB Mail Enumeration + IMAP Email Flag Retrieval — Full End-to-End Summary

## Objective
Enumerate mail services on `10.129.1.2`, authenticate with IMAP using `robin:robin`, locate internal mailboxes, read the stored email, and extract the **flag contained in the email body**.

---

## 1. Initial Service Discovery
Nmap revealed mail services running on the target:

- **110/tcp** → POP3 (`Dovecot pop3d`)
- **993/tcp** → IMAPS (IMAP over TLS)
- **995/tcp** → POP3S (POP3 over TLS)

POP3 capabilities included:
`CAPA PIPELINING STLS UIDL RESP-CODES SASL TOP AUTH-RESP-CODE`

TLS certificate enumeration showed:
- `CN=dev.inlanefreight.htb`
- `emailAddress=cto.dev@dev.inlanefreight.htb`
- Self-signed certificate with long validity

This confirmed an internal mail infrastructure using the `inlanefreight.htb` domain.

---

## 2. IMAPS Enumeration with curl (Banner + Auth Validation)
You connected to IMAPS using curl:

```bash
curl -k imaps://10.129.1.2 --user robin:robin -v
```
Results:
```
TLS handshake succeeded
Authentication succeeded
IMAP server greeting banner displayed a flag:
HTB{roncfbw7iszerd7shni7jr2343zhrj}
```

Mailbox listing revealed:
```
INBOX
DEV
DEV.DEPARTMENT
DEV.DEPARTMENT.INT
This confirmed:
Credentials are valid
IMAP is accessible
The interesting mailbox is not INBOX
```

3. IMAP Interactive Session (Protocol-Correct Enumeration)
You switched to an interactive IMAP session using OpenSSL:

```bash
openssl s_client -connect 10.129.1.2:993 -crlf -quiet
```
Then executed proper IMAP commands:
```
A1 LOGIN robin robin
A2 LIST "" *
```
This again confirmed the mailboxes:
```
INBOX
DEV.DEPARTMENT.INT
```
4. Inbox Check (Empty)
You selected the INBOX:
```
A3 SELECT INBOX
Response showed:
0 EXISTS
```
Meaning no emails were stored in INBOX.

5. Selecting the Internal Department Mailbox
You correctly selected the internal mailbox:
```bash
A4 SELECT DEV.DEPARTMENT.INT
```
Response showed:
1 EXISTS
```
Meaning one email is present in this mailbox.

6. Reading the Email Header
You fetched the email headers:
```
A5 FETCH 1 BODY[HEADER]
Header contents:
yaml
Copy code
Subject: Flag
To: Robin <robin@inlanefreight.htb>
From: CTO <devadmin@inlanefreight.htb>
Date: Wed, 03 Nov 2021 16:13:27 +0200
This revealed:
Internal admin email address: devadmin@inlanefreight.htb
The email is explicitly titled “Flag”
```
7. Reading the Email Body (Actual Flag)
You then fetched the email body:
```A6 FETCH 1 BODY[TEXT]
Email body contained the actual required flag:
HTB{983uzn8jmfgpd8jmof8c34n7zio}```
8. Final Result
The correct flag retrieved from inside the IMAP email body is:
```HTB{983uzn8jmfgpd8jmof8c34n7zio}
Key Takeaways
Banner flags ≠ task flags (HTB often differentiates)
```

INBOX may be empty; enumerate all mailboxes
Use protocol-correct commands (IMAP ≠ POP3)
Internal mailboxes often contain sensitive data
Fetching BODY[TEXT] is required to read actual message content

**SNMP**
The SNMP module explains how the Simple Network Management Protocol is used to monitor and manage networked devices such as routers, switches, servers, and IoT systems. SNMP operates primarily over UDP port 161 for queries and configuration changes, while traps are sent asynchronously from servers to clients over UDP port 162 when specific events occur. Central to SNMP is the Management Information Base (MIB), a standardized, hierarchical structure written in ASN.1 that defines Object Identifiers (OIDs). OIDs uniquely identify pieces of information on a device and allow clients to query system details in a consistent way across vendors.

The module compares SNMP versions, highlighting that SNMPv1 and SNMPv2c lack encryption and rely on plaintext community strings for access control, making them insecure and attractive targets for attackers. SNMPv3 significantly improves security through authentication and encryption but is more complex to configure, which leads many organizations to continue using older versions.

From an offensive perspective, the module demonstrates how misconfigurations—such as default or weak community strings and read/write access—can expose sensitive system information. Tools like snmpwalk, onesixtyone, and braa are used to enumerate OIDs, brute-force community strings, and extract data such as hostnames, installed software, and administrator contact details. The module emphasizes that SNMP can be both a powerful administrative tool and a serious security risk when improperly configured.

SNMP walk was able to enumerate the version of SNMP, the admin contact and a custom script that was running on a server. Very powerfull enumeration tool.

```bash
snmpwalk -v2c -c public 10.129.192.69
```