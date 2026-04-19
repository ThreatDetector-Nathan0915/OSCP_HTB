# FTP anonymous access (walkthrough)

Anonymous **FTP** allows unauthenticated read (sometimes write) access—common in legacy labs and a **high-impact** finding if data is sensitive.

---

## Step 1 — Nmap

**Command:**

```bash
nmap -sV -sC -O 10.129.201.243
```

| Flag | Purpose |
|------|---------|
| `-sV` | Version detection |
| `-sC` | Default NSE scripts (often report `ftp-anon`, banner text) |
| `-O` | OS fingerprint (may be blocked or inaccurate; adds packets) |

**Finding:** **TCP 21** — FTP with **anonymous login allowed** (confirm exact banner/script output in your scan).

---

## Step 2 — Connect with `ftp`

**Client:**

```bash
ftp 10.129.201.243
```

**Credentials:**

- Username: `anonymous`
- Password: usually **guest e-mail** or **blank** (press Enter)—follow server prompt text.

**Inside the client:**

```text
ls          # list remote directory
get flag.txt   # download file to local CWD
```

| Command | Purpose |
|---------|---------|
| `ls` | List remote files (implementation may use `LIST` or `NLST`) |
| `get <file>` | Download a single file |

---

## Takeaways

- Anonymous FTP is a **misconfiguration** when the share contains non-public data.
- Nmap **`-sC`** frequently surfaces anonymous FTP without manual banner grabs.
- Prefer **`lftp`** or **`curl -u anonymous:`** for scripting; classic `ftp` is fine for quick labs.

---

## Summary

| Step | Command | Note |
|------|---------|------|
| Scan | `nmap -sV -sC -O 10.129.201.243` | Confirms FTP + scripts |
| Connect | `ftp 10.129.201.243` | `anonymous` / blank |
| Retrieve | `get flag.txt` | Saves locally |

This pattern illustrates **exposed legacy services**—always verify whether anonymous read/write is intended before closing a finding.
