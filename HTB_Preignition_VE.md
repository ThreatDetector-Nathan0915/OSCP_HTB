# Gobuster and weak default credentials

Directory discovery with **Gobuster** plus **`admin` / `admin`** on a hidden admin endpoint—a common training pattern for **default credential** and **sensitive path** failures.

---

## 1. Port scan

**Command:**

```bash
nmap -sV 10.129.121.8
```

| Flag | Purpose |
|------|---------|
| `-sV` | Service and version detection on each open port |

**Finding:** **TCP 80** — HTTP service.

**QC:** For unknown scope, start with `-Pn` if ICMP is filtered, or a top-port scan before full `-p-`.

---

## 2. Directory brute force

**Command:**

```bash
gobuster dir -u http://10.129.121.8 -w /usr/share/wordlists/dirb/common.txt
```

| Flag | Purpose |
|------|---------|
| `dir` | HTTP path discovery mode |
| `-u` | Base URL (include `http://` or `https://`) |
| `-w` | Wordlist file |

**Finding:** **`/admin.php`** returned **HTTP 200** — manually review in browser (not every 200 is a real admin panel; some apps return soft 404s).

---

## 3. Default credentials

At `http://10.129.121.8/admin.php`, the login form accepted:

```text
Username: admin
Password: admin
```

**Lesson:** enforce password policy, disable unused admin accounts, and protect admin paths with **network controls** and **MFA** where applicable.

---

## 4. Post-login

Complete the lab objectives (flag submission, configuration review, or hardening tasks as specified by HTB).

---

## Summary

| Step | Command / action | Result |
|------|------------------|--------|
| Scan | `nmap -sV 10.129.121.8` | HTTP on port 80 |
| Content discovery | `gobuster dir -u http://10.129.121.8 -w …/common.txt` | `/admin.php` |
| Auth | `admin` / `admin` | Authenticated admin session |

---

## Takeaways

- **Gobuster** (or `feroxbuster`, `ffuf`) quickly surfaces hidden PHP/HTML routes.
- **Default credentials** remain a valid test case on appliances and internal apps.
- Always record **HTTP status**, **title**, and **response length** to avoid false positives.

**Status:** lab objectives completed.
