# Just General Notes from the Pentration tester path. Note dump really.
---
## Common Ports
20/21   (TCP)	    FTP File Transfer Protocol, used to move files around
22      (TCP)	    SSH Secure Shell, used to securly connect to and manage hosts
23      (TCP)	    Telnet Same as SSH just not secure at all, used to connect to hosts and manage them just in clear text.
25      (TCP)	    SMTP Simple Mail Tranfer Protocol used to send and recieve mail for mail servers.
80      (TCP)	    HTTP Hypertext Transfer Protocol used to tranfer web content between client and a web server.
161     (TCP/UDP)	SNMP Simple Network Management Protocol used to monitor, manage and configure network devices - pull metrics - network invetory ect.
389     (TCP/UDP)	LDAP Lightweight Directory Access Protocol - used to query directory services, like active directory.
443     (TCP)	    SSL/TLS (HTTPS) Hypertext Transfer Protocol Secure - encypted secure version of HTTP. used for loading websites ect.
445     (TCP)	    SMB Server Message Block network local file sharing, people stand up and SMB drive and others connect to it and pull files.
3389    (TCP)	    RDP Remote Desktop Protocol - Like SSH but has an interactive session that you can interact with the GUI of the host.

### Different Tools
**SSH** - Can be configered for password or passwordless configuration, with public-key authentication. Uses a client server model. IE the host needs to be running OpenSSH or an SSH server.
Usage 
```bash
ssh bob@10.10.10.10
```
**Netcat** - Netcat, is a networking utility. iAnd is great for connecting shells in a pen test. You can use it to either catch a connection or send one. Can also be used for banner grabbing which basically you point it at a port on the target host and it will return the service running on that port. 
```bash
nc 10.10.10.10 22
```
**returns OpenSSH running on 22**

**Tmux** - Used for multiboxing terminals, ctr + b opens a new terminal, and we can swtich between them with 0, 1 ect. Shift + " will split them horizontally, and Shift + % will split them vertically. Also can switch between the windows with up down left or right keypad arrows.

**VIM** - a text editor.. to edit file hit i (insert mode) 
Command	Description
x	    Cut character
dw	    Cut word
dd	    Cut full line
yw	    Copy word
yy	    Copy full line
p	    Paste
:1	    Go to line number 1.
:w	    Write the file, save
:q	    Quit
:q!	    Quit without saving
:wq	    Write and quit

**Ports** - range from 1 to 65,535 well known ports are 1 through 1023, port 0 is treted as a wild card port.

**NMAP** - Used to scan hosts in a subnet and enumerate services operating on ports. Just doing --
```bash
namp <target_ip>
```
will run a quick scan against the 1000 most common ports returning the state, service, port, ect. We can use the -sC parameter to define nmap common/default scripts usage. the -sV flag will identify the service and version from the scan. -p- will tell nmap to scan all 1-65,000 ports (all ports) this will take alot longer than just a standard scan.
    NMAP scripts in general can extend the utility of NMAP notably. To find them we can run
```bash
locate scripts/<script_you_want>
```
```bash
nmap --script <script_name> -p<ports> <host>
```

**FTP** - File transfer protocol can be used to pull files from a host if left open with anonymous login allowed.
usage
```bash
ftp -p 10.129.42.253
get login.txt
```
**SMB** - Server Message Block can actually be used for RCE via eternal blue, but for the most part another good vector to find sensitive files and passwords, like seen in ftp. smb-os-disovery.nse can be used to extract the OS running on a host, can be usefull. To login to an smv share the following works for listing all the shares on a client
```bash
smbclient -N -L \\\\10.129.53.353
```
that will also list the users and the shares they are a part of. To target a users share run.
```bash 
smbclient \\\\10.129.42.253\\users
```
If you need to do it in the contex of a user the following works but also will prompt for a password.
```bash
smbclient -U bob \\\\10.129.43.253\users
```
---
#### **Web Eumeration**

**Gobuster** - Versatile tool that can perform DNS, vhost, and directory brute-forcing. Can also enumerate public AWS S3 buckets. 
Usage - directory bruteforce mode.
```bash
gobuster dir -u http://10.10.10.121/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```
Link to http status codes: https://en.wikipedia.org/wiki/List_of_HTTP_status_codes

