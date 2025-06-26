# 🗂️ Gobuster + Weak Login – Simple Web Exploit

This box was a textbook example of directory brute-forcing combined with weak credential hygiene.

---

## 🔍 1. Nmap Enumeration

Began with a version and service scan:

```bash
nmap -sV 10.129.121.8
```

**Findings:**
- Port `80/tcp` open — running HTTP web server

---

## 📂 2. Web Directory Enumeration with Gobuster

Used `gobuster` to brute force accessible directories:

```bash
gobuster dir -u http://10.129.121.8 -w /usr/share/wordlists/dirb/common.txt
```

**Result:**
- Found `/admin.php` – returned a **200 OK** response

---

## 🔐 3. Default Credentials Exploit

Navigated to `http://10.129.121.8/admin.php`, which presented a login form.

Tried default web credentials:

```text
Username: admin
Password: admin
```

**It worked.** Immediate access to the admin panel.

---

## 🏁 4. Post-Login

Once inside the panel:
- Retrieved flag or executed post-auth steps (depending on the lab)

---

## ✅ Summary

| Step               | Command                                                              | Result                            |
|--------------------|----------------------------------------------------------------------|-----------------------------------|
| Nmap Enum          | `nmap -sV 10.129.121.8`                                              | Found HTTP on port 80             |
| Gobuster Scan      | `gobuster dir -u http://10.129.121.8 -w common.txt`                 | Found `/admin.php`                |
| Web Login Attempt  | Used `admin:admin`                                                   | Successfully authenticated        |

---

## 🧠 Lessons Learned

- **Gobuster** is critical for catching hidden pages.
- **Default creds** are still dangerous and commonly overlooked.
- Check status codes — 200s are always worth a visit.

**Box complete. Easy and clean. ✅**
