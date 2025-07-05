# 🔍 Hack The Box - `nocturnal.htb` Writeup  
**Difficulty**: Medium  
**IP Address**: `10.129.232.23`  
**Hostname**: `nocturnal.htb`  
**Author**: Your relentless enumeration 😤💪  

---

## 🌐 Initial Recon - Nmap All The Things!

```bash
nmap -sC -sV -oN nmap/initial 10.129.232.23
```

**Results:**
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu
80/tcp open  http    nginx 1.18.0 Ubuntu
```

⏩ Port 80 auto-redirects to `http://nocturnal.htb/`  
🧠 Add to `/etc/hosts`:

```bash
echo "10.129.232.23 nocturnal.htb" | sudo tee -a /etc/hosts
```

---

## 🕸️ Web Enumeration - Portal Discovery!

Browsing to `http://nocturnal.htb/` gives us a **file upload login portal**.

🪪 Login discovered via guesswork:

```text
Username: user
Password: user
```

✅ Successful login reveals an upload form.

📝 After upload, files are available at:

```
http://nocturnal.htb/view.php?username=user&file=<filename>
```

---

## 📂 File Upload Analysis - Let the Shell Games Begin!

Allowed extensions (as per error message):

```text
Invalid file type. pdf, doc, docx, xls, xlsx, odt are allowed.
```

### 🔥 Attempted Payloads

| Payload                        | Result                                              |
|-------------------------------|-----------------------------------------------------|
| `shell.php.doc`               | Upload successful, but no execution 😤              |
| `shell.php%00.doc`            | Server responds, but no null byte behavior observed |
| `?file=shell.php.doc`         | File shown as plain text – PHP not parsed           |
| `?file=shell.doc`             | Same result                                         |
| `?file=shell.php%250.doc`     | No execution, `%250` ineffective                    |

### 🔬 Theory:

- PHP is **not parsed**, likely due to Nginx config treating uploads as static files.
- `view.php` appears to **read** and **stream** files, not include/execute.
- Extension restriction bypass attempts (null byte, double extensions) all failed.

---

## 📁 Directory Enumeration - Hidden Paths Discovered!

### 🔎 Directory Brute-force with FFUF:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u "http://nocturnal.htb/FUZZ" -r
```

🧾 Notable Results:

```
uploads     [Status: 403]
uploads2    [Status: 403]
backups     [Status: 403]
```

🔐 Forbidden — confirms those dirs **exist** but are protected by Nginx or file perms.

---

## 🌐 Subdomain Fuzzing - Expanding the Surface

```bash
wfuzz -c -w subdomains.txt -u 'http://nocturnal.htb/' -H "Host: FUZZ.nocturnal.htb" --hw 10
```

⚠️ No valid subdomains returned — all 200 responses were default page sizes.

---

## 🧩 Current Summary

### ✅ Confirmed:
- Uploads are saved to a user-specific directory
- Uploads are accessible via `view.php` but only rendered/downloaded — **not executed**
- Extensions restricted to doc-like types (`.pdf`, `.doc`, etc.)
- Directories like `/uploads`, `/uploads2`, `/backups` exist but are **403 Forbidden**
- No working RCE or LFI behavior yet
- Subdomain fuzzing yielded no results

### ❌ Tried and Failed:
- `shell.php.doc` → no execution
- Double extensions, null byte injection, `%250` tricks
- Subdomain brute-forcing
- Direct access to `/uploads` — forbidden

---

## 🧠 Hypothesis Going Forward

### Potential Exploit Vectors to Investigate:

1. **File Parsing Exploit**
   - Upload malformed `.docx`, `.pdf` etc. with embedded payloads.
   - Leverage tools like `Burp`, `Metasploit`, or `officeparser` for OLE/Docx macro injection.

2. **SSRF / LFI via `view.php`**
   - Try accessing internal resources via:
     ```http
     /view.php?username=user&file=../../../../../../etc/passwd
     /view.php?username=admin&file=secret.txt
     /view.php?username=../uploads2&file=shell.doc
     ```
   - Maybe `view.php` has weak path sanitization.

3. **File Overwrite**
   - Attempt file overwrite on backend — is `view.php` using real path with `username`/`file` directly?

4. **Upload Triggers (Cron / Parser)**
   - Maybe another process (like an AV scanner or cron job) parses uploaded documents — try:
     - **.pdf with JavaScript**
     - **.docx with macro**

5. **Monitor for Time-Based Execution**
   - Upload payloads, then monitor traffic for delayed callback (watch `tcpdump` on tun0).

---

## 🔚 Final Thoughts (for now)

You've fully mapped the front door — login, upload, directory structure. The next step is **deeper exploitation via document payloads**, LFI edge cases, or back-end parser triggers. This box is taunting you with a classic **"uploads, but no execution"** trap!

🧠 Stay relentless — your shell awaits... 🐚🔥
