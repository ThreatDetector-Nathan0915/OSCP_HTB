# Web Exploitation Walkthrough – Host 192.168.59.232

This walk-through shows how we went from basic enumeration → discovering an exposed PDF generator → fingerprinting it as **mPDF 6.0**, which is known to have vulnerabilities.

---

## Step 1: Full Port Scan

Ran an all-ports scan to identify exposed services:

```bash
nmap -p- 192.168.59.232 -vv
```

### Output Summary:
```text
22/tcp     open     ssh
80/tcp     open     http
10000/tcp  open     snet-sensor-mgmt
```

> Port 80 is of immediate interest as it typically serves a web app.

---

## Step 2: Install FFUF for Web Directory Enumeration

Installed FFUF and a wordlist to begin fuzzing:

```bash
sudo apt update
sudo apt install ffuf
```

### Download SecLists Wordlist:

```bash
wget -O SecLists.zip https://github.com/danielmiessler/SecLists/archive/refs/heads/master.zip
unzip SecLists.zip
```

---

## Step 3: Directory Fuzzing on Port 80

Navigated to the SecLists directory and ran FFUF:

```bash
cd SecLists-master/Discovery/Web-Content
ffuf -w common.txt -u "http://192.168.59.232/FUZZ" -e .php
```

### FFUF Discovery:

- `/config/` directory found
- Inside it: `config.php`

So the valid page becomes:

```text
http://192.168.59.232/config/config.php
```

---

## Step 4: Analyzing the Web App

The main site functions as a **PDF converter**. Likely vulnerable if using an outdated library.

### Use `exiftool` to Inspect Generated PDF

Downloaded a sample PDF and checked its metadata:

```bash
exiftool mpdf.pdf
```

### Output:
```text
Producer: mPDF 6.0
...
```

**mPDF 6.0 identified** as the PDF generation backend.

---

## Step 5: Look for Known Vulnerabilities in mPDF 6.0

Now that we’ve fingerprinted the PDF generation engine, we can research public exploits or known vulnerabilities.

> Common issues with older mPDF versions:
- **XSS injection in PDF templates**
- **Arbitrary file read via template misconfig**
- **Local File Inclusion (LFI)**
- **Remote Code Execution (RCE) under certain PHP settings**

---

## Next Steps

1. Search Exploit-DB and GitHub for **mPDF 6.0 vulnerabilities**
2. Attempt **payload injection** in fields like headers or PDF body
3. Test for **template manipulation** (e.g., Smarty or PHP wrappers)
4. Look into directory traversal or file inclusion possibilities via the upload fields

---

## Summary

| Phase            | Tool/Action                               | Result                        |
|------------------|--------------------------------------------|-------------------------------|
| Port Scan        | `nmap -p-`                                 | Found SSH, HTTP, SNET         |
| Web Enum         | `ffuf + SecLists`                         | Found `/config/config.php`    |
| PDF Inspection   | `exiftool`                                | Identified `mPDF 6.0`         |
| Vulnerability Check | Manual / Exploit-DB search             | Prepping for RCE or LFI chain |

---

Next: weaponize the mPDF issue or build converter-specific payloads according to the lab brief and legal scope.
