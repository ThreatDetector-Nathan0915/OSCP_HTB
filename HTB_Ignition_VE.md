# Hack The Box Walkthrough: *Ignition*  
>  Objective: Bypass weak authentication on a Magento admin portal and escalate to full system access!

---

## STEP 1 — Initial Recon with Nmap

Scan the target for open ports and default services.

```bash
nmap -sV -sC 10.129.99.94
```

```text
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-03 22:34 EDT
Nmap scan report for 10.129.99.94
Host is up (0.048s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
|_http-title: Did not follow redirect to http://ignition.htb/
|_http-server-header: nginx/1.14.2
```

 The site redirects to `http://ignition.htb/`. Let's map that in our `/etc/hosts`.

---

## STEP 2 — Hostname Mapping

Edit `/etc/hosts` and add the following entry:

```bash
sudo nano /etc/hosts
```

Add:

```text
10.129.99.94 ignition.htb
```

Now the domain will resolve locally. Let’s check it out in the browser or use `curl`:

```bash
curl -I http://ignition.htb
```

---

## STEP 3 — Directory Enumeration with Ffuf

Run directory enumeration to locate hidden paths:

```bash
ffuf -w medium.txt -u "http://ignition.htb/FUZZ"
```

 Result:

```text
admin               [Status: 200, Size: ...]
```

Enumeration surfaced an **`/admin`** route (HTTP 200).

---

## STEP 4 — Investigate the Admin Portal

Visiting `http://ignition.htb/admin` shows a login page with **Magento** branding.

Magento is a PHP e-commerce stack; weak default or reused admin credentials are a common finding on internal labs.

 Password requirements: minimum **7 characters**, **letters and numbers**.

---

## STEP 5 — Weak Credential Testing

Manual testing with common usernames and passwords like:

- `admin:admin`
- `admin:password123`
- `admin:qwerty123`

**Result:** valid admin credentials.

```text
Username: admin
Password: qwerty123
```

Authenticated access to the Magento admin dashboard is confirmed.

---

## NEXT STEPS

From here, you could:

- Check for known Magento RCE exploits (e.g., CVE-2015-1397, CVE-2016-4010)
- Upload a malicious extension or theme
- Abuse admin access to execute code via cron templates or product attributes

Next steps typically include reviewing known Magento admin abuse chains (CVEs, extension or theme upload, dangerous cron templates) under lab rules.

---
Initial foothold achieved; proceed to privilege escalation or lateral movement as required by the exercise.
