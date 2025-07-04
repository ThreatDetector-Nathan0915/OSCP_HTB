# 🚀 Hack The Box Walkthrough: *Ignition*  
> 🎯 Objective: Bypass weak authentication on a Magento admin portal and escalate to full system access!

---

## 🔍 STEP 1 — Initial Recon with Nmap

Let's scan the box for open ports and default services.

```bash
nmap -sV -sC 10.129.99.94
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-03 22:34 EDT
Nmap scan report for 10.129.99.94
Host is up (0.048s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
|_http-title: Did not follow redirect to http://ignition.htb/
|_http-server-header: nginx/1.14.2
```

💡 The site redirects to `http://ignition.htb/`. Let's map that in our `/etc/hosts`.

---

## 🗺️ STEP 2 — Hostname Mapping

Edit `/etc/hosts` and add the following entry:

```bash
sudo nano /etc/hosts
```

Add:

```
10.129.99.94 ignition.htb
```

Now the domain will resolve locally. Let’s check it out in the browser or use `curl`:

```bash
curl -I http://ignition.htb
```

---

## 🧪 STEP 3 — Directory Enumeration with Ffuf

Let’s fuzz for hidden paths to find something juicy:

```bash
ffuf -w medium.txt -u "http://ignition.htb/FUZZ"
```

🎯 Result:

```
admin               [Status: 200, Size: ...]
```

Boom! We’ve discovered an `/admin` portal!

---

## 🕵️ STEP 4 — Investigate the Admin Portal

Visiting `http://ignition.htb/admin` shows a login page — and... 🧩 the **Magento logo** is present!

🧠 Magento is a PHP-based e-commerce platform. Time to brute smarter, not harder.

📜 Password requirements: minimum **7 characters**, **letters and numbers**.

---

## 🔐 STEP 5 — Weak Credential Testing

Manual testing with common usernames and passwords like:

- `admin:admin`
- `admin:password123`
- `admin:qwerty123`

🎉 Success!

```text
Username: admin
Password: qwerty123
```

💥 We're in the Magento admin dashboard!

---

## 🔭 NEXT STEPS

From here, you could:

- Check for known Magento RCE exploits (e.g., CVE-2015-1397, CVE-2016-4010)
- Upload a malicious extension or theme
- Abuse admin access to execute code via cron templates or product attributes

Let me know when you're ready to go full RCE mode, and we’ll blow the backdoor open! 🧨

---
🏴‍☠️ Initial foothold achieved. Time to escalate! 💪
