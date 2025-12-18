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