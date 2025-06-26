# 📦 FTP Credential Reuse & Web Login – HTB Walkthrough

---

## 🔍 Initial Nmap Enumeration

```bash
nmap -sV -sC 10.129.155.217
```

- Found open FTP port (21) with anonymous login allowed.
- Port 80 (HTTP) hosting a basic web server.

---

## 📁 FTP Enumeration & Credential Extraction

```bash
ftp 10.129.155.217
```

- Logged in using:
  ```
  Name: anonymous
  ```
- Listed directory contents:
  ```bash
  ftp> dir
  ```
- Downloaded files of interest:
  ```bash
  ftp> get allowed.userlist
  ftp> get allowed.userlist.psswd
  ```

### 🔍 Local inspection of files:

```bash
cat allowed.userlist
cat allowed.userlist.psswd
```

- Revealed usernames and passwords that could be reused elsewhere.

---

## 🌐 Web Server Exploration

Visited `http://10.129.155.217` in browser — saw a basic, non-functional landing page.

### Directory Enumeration with Gobuster

```bash
gobuster dir --url http://10.129.155.217/ -w common.txt -x php,html
```

- `-x php,html` ensures Gobuster only checks `.php` and `.html` file extensions.
- Discovered `/login.php`.

---

## 🔐 Credential Reuse Attempt

- Navigated to: `http://10.129.155.217/login.php`
- Tried username/password combos from the FTP dump manually.
- Eventually succeeded in logging in.

---

## 🏁 Final Result

- Logged into web portal using reused FTP credentials.
- Flag was retrieved from the authenticated area.
- Box pwned using FTP to Web login pivot 💥

```bash
✔️ Anonymous FTP access
✔️ Credential extraction
✔️ Gobuster for page discovery
✔️ Web login using reused creds
✔️ Flag captured
```
