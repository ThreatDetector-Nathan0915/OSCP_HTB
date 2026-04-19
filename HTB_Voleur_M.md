# Voleur — enumeration notes (AD + SSH)

Two-phase **Nmap** workflow: fast **full TCP SYN** sweep to find open ports, then a **targeted** `-sC -sV` pass on the discovered set for banners and scripts. Raw transcript below is **QC’d** for typos in the original manual port list (`539` corrected to **`593`** where applicable).

---

## Phase A — Full TCP SYN scan

**Command:**

```bash
sudo nmap -p- -sS --min-rate 5000 -n -Pn 10.129.238.152
```

| Flag | Purpose |
|------|---------|
| `-p-` | All TCP ports |
| `-sS` | **SYN** half-open scan (requires raw packets → often needs `sudo`) |
| `--min-rate 5000` | Minimum packet rate (faster; noisier—reduce on fragile networks) |
| `-n` | Skip reverse-DNS resolution |
| `-Pn` | Skip host discovery (treat host as up—use when ICMP is filtered) |

**Observations from output:** classic **AD** surface (**53, 88, 135, 139, 389, 445, 464, 636, 3268–3269, 5985, 9389, …**) plus **SSH** on **2222/tcp** (non-default).

---

## Phase B — Version + scripts on selected ports

**Command:**

```bash
sudo nmap -sC -sV -p 53,88,135,139,389,445,464,593,636,2222,3268,3269,5985,9389,49664,49668,49670,49671,52699,53123,53142 10.129.238.152
```

| Flag | Purpose |
|------|---------|
| `-sC` | Default safe scripts (SMB signing, time, etc.) |
| `-sV` | Version detection |
| `-p ...` | Comma list built from phase A (omit closed/filtered to save time) |

**QC note:** If your saved command used **`539`**, replace with **`593`** (`http-rpc-epmap`) unless your transcript genuinely showed 539.

---

## Raw output (reference)

```text
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-10 20:31 EDT
...
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
2222/tcp  open  EtherNetIP-1
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
...
```

```text
Nmap scan report for dc.voleur.htb (10.129.238.152)
...
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
2222/tcp  open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
...
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required
```

**Interpretation hints:**

- **LDAP/Kerberos/445/5985/9389** → domain controller footprint; plan **LDAP**, **SMB**, and **WinRM** enumeration next.
- **`Message signing enabled and required`** → relay attacks against SMB signing **blocked**; pivot to other paths (Kerberos, LDAP, PKINIT, etc.) per lab.
- **SSH on 2222** → possible **dual-homed** or management service; verify with `ss -tulnp` equivalent on host if you gain access.

---

## Next commands (not run in log)

Typical follow-ups (scope-dependent):

```bash
enum4linux-ng -A 10.129.238.152
ldapsearch -x -H ldap://10.129.238.152 -s base namingcontexts
nxc smb 10.129.238.152
```

Document outputs in separate sections as you progress.
