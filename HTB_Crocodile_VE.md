# FTP credential reuse and web login

Anonymous **FTP** exposes credential files; the same material often unlocks the **web** application.

---

## Nmap

```bash
nmap -sV -sC 10.129.155.217
```

| Flag | Purpose |
|------|---------|
| `-sV` | Service/version |
| `-sC` | Default NSE scripts (often flags anonymous FTP) |

**Typical findings:** **TCP 21** (FTP), **TCP 80** (HTTP).

---

## FTP — anonymous login

```bash
ftp 10.129.155.217
```

Use username **`anonymous`** (blank or guest e-mail password if prompted).

**Useful FTP client commands:**

```text
dir
get allowed.userlist
get allowed.userlist.psswd
```

**Local review:**

```bash
cat allowed.userlist
cat allowed.userlist.psswd
```

Extract username/password pairs for reuse testing.

---

## Web and Gobuster

Browse `http://10.129.155.217`. Static landing pages often hide a real app behind a path.

**Directory brute force:**

```bash
gobuster dir --url http://10.129.155.217/ -w /usr/share/wordlists/dirb/common.txt -x php,html
```

| Flag | Purpose |
|------|---------|
| `-x php,html` | For each wordlist entry, also try **`.php`** and **`.html`** suffixes (helps find `login.php`, `index.html`, etc.) |

**QC:** Ensure `-w` points to a real path on your system (`common.txt` alone is rarely correct—prefer full path as in Preignition notes).

---

## Credential reuse

Open `http://10.129.155.217/login.php` (or path discovered by Gobuster) and attempt combinations from the FTP files until login succeeds.

---

## Summary checklist

- Anonymous FTP → download user/password lists  
- Gobuster → discover `login.php`  
- Reuse credentials → authenticated area → flag  

**Status:** initial access via FTP; pivot to web application; objectives completed.
