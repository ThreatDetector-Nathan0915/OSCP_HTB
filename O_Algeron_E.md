# 🧭 Web Recon Walkthrough – Host 192.168.191.65

---

## 🔍 Step 1: Initial Service Enumeration

Performed a version scan using Nmap:

```bash
sudo nmap -sV 192.168.191.65
```

### Output:
```
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
80/tcp   open  http          Microsoft IIS httpd 10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
9998/tcp open  http          Microsoft IIS httpd 10.0
```

**Target is a Windows server running IIS.**

---

## 🌐 Step 2: Web Interface Investigation

Browsed to both web ports in Firefox:

- **Port 80**: Blank or default page.
- **Port 9998**: Web login interface detected.

---

## 🛡️ Step 3: Vulnerability Scan (NSE)

Ran Nmap NSE scripts against known services but **no critical vulnerabilities** were returned:

```bash
nmap --script vuln 192.168.191.65
```

---

## ❌ Step 4: Handling False Positives (404s Masked as 200s)

During brute force attempts, noticed that invalid paths return:

- **HTTP 200 OK**
- But they are actually **404 errors** disguised with custom error page

This breaks most brute force tools that rely on real `404 Not Found` responses.

---

## ✅ Step 5: FFUF to the Rescue

Used `ffuf` with status code matching and forced size filtering to bypass the masked 404s:

```bash
ffuf -u "http://192.168.191.65:9998/Interface/errors/404.html?aspxerrorpath=/FUZZ" \
-w raft-medium-files.txt -mc 200 -t 40 -fs 4845
```

### Explanation:
- `-u`: Target URL, using the server’s error handler
- `-w`: Wordlist to brute-force paths (`raft-medium-files.txt`)
- `-mc 200`: Only show responses with status code 200
- `-fs 4845`: Filter out responses that always return the same length (false 404s)
- `-t 40`: Threading to speed up discovery

---

## 📌 Summary

| Phase        | Tool     | Result |
|--------------|----------|--------|
| Port Scan    | `nmap -sV` | Identified IIS & FTP on Windows |
| Web Recon    | Firefox  | Port 9998 served a login page |
| Vuln Scan    | Nmap NSE | No major issues detected |
| Fuzzing      | ffuf     | Bypassed masked 404 pages using length filtering |

---

## 🔜 Next Steps

- Analyze `ffuf` results for interesting endpoints
- Test for file upload, RCE, or default credentials on login
- Check FTP for anonymous access or weak creds:
  ```bash
  ftp 192.168.191.65
  ```

