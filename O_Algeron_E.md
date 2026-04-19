# Web reconnaissance — host `192.168.191.65`

Windows target with **IIS**, duplicate HTTP services, and **soft 404** behavior that breaks naive directory brute-forcing. Below: service map, browser recon, NSE scripts, and **`ffuf`** calibration.

---

## Step 1 — Service enumeration

**Command:**

```bash
sudo nmap -sV 192.168.191.65
```

| Detail | Purpose |
|--------|---------|
| `sudo` | May be required for certain scan types; here used for consistency with raw socket workflows |
| `-sV` | Version detection for open ports |

**Sample interpretation:**

```text
21/tcp   ftp      Microsoft ftpd
80/tcp   http     Microsoft IIS 10.0
135/tcp  msrpc
139/tcp  netbios-ssn
445/tcp  microsoft-ds
9998/tcp http     Microsoft IIS 10.0
```

**Finding:** Windows server with **IIS** on **80** and **9998**.

---

## Step 2 — Manual web review

- **Port 80:** default or blank IIS page.
- **Port 9998:** login / application UI — primary manual test surface.

---

## Step 3 — Nmap NSE (`vuln` category)

```bash
nmap --script vuln 192.168.191.65
```

**Purpose:** runs **vulnerability-oriented** NSE scripts where they apply (SMB HTTP, etc.). **QC:** output can include **false positives**—correlate with version numbers and manual validation.

**Result (this host):** no high-confidence critical issues from scripts alone.

---

## Step 4 — Soft 404 (HTTP 200 masking missing pages)

Invalid paths may return **HTTP 200** with a **fixed-size** error template. Tools that assume “404 means not found” will report **noise hits**.

**Mitigation:** filter on **response length**, **word count**, or **regex** of stable template markers.

---

## Step 5 — `ffuf` with size filter

**Command:**

```bash
ffuf -u "http://192.168.191.65:9998/Interface/errors/404.html?aspxerrorpath=/FUZZ" \
  -w raft-medium-files.txt -mc 200 -t 40 -fs 4845
```

| Flag | Role |
|------|------|
| `-u` | URL; `FUZZ` marks injection point; here the app surfaces missing paths via `aspxerrorpath` |
| `-w` | Wordlist (`raft-medium-files.txt` or SecLists equivalent) |
| `-mc 200` | Only display HTTP 200 responses |
| `-fs 4845` | **Hide** responses of size 4845 bytes (calibrated soft-404 body) |
| `-t 40` | Concurrency |

**QC:** Run **`ffuf` once with a known-bad path** to measure the decoy response size, then set `-fs` (or combine with `-fw` / regex filters per `ffuf` docs).

---

## Summary

| Phase | Tool | Outcome |
|-------|------|---------|
| Ports | `nmap -sV` | IIS + auxiliary services |
| Browser | Manual | Login UI on **9998** |
| Scripts | `nmap --script vuln` | No immediate critical hits |
| Content discovery | `ffuf` | Bypass soft-404 using response size |

---

## Next steps

- Mine `ffuf` results for real ASPX/ASHX/AXD endpoints.
- Test authentication, file upload, and deserialization issues on **9998**.
- Enumerate **FTP** (`anonymous`, weak creds, writable `wwwroot`):

```bash
ftp 192.168.191.65
```

Use `binary` / `passive` as needed; log all actions for reporting.
