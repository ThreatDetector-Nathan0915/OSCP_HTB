# 📁 FTP Anonymous Access – Full Walkthrough

This writeup outlines a simple exploitation scenario involving anonymous FTP access, starting with basic enumeration and leading to successful file retrieval.

---

## 🔍 Step 1: Nmap Scan for Enumeration

We used a comprehensive `nmap` scan with service detection, default scripts, and OS guessing:

```bash
nmap -sV -sC -O 10.129.201.243
```

### 🔎 Flags Explained:
- `-sV` → Detect service versions
- `-sC` → Run default NSE scripts for more info
- `-O` → Attempt OS fingerprinting

---

## ✅ Step 2: FTP Discovered with Anonymous Login

The scan output revealed:

- **Port:** `21/tcp`
- **Service:** `ftp`
- **Access:** `Anonymous login allowed`

> Anonymous access means anyone can connect to the FTP server without credentials, typically using `anonymous` as the username.

---

## 📂 Step 3: Accessing the FTP Server

Connect using the `ftp` client:

```bash
ftp 10.129.201.243
```

When prompted for credentials:
- **Username:** `anonymous`
- **Password:** *press Enter*

Once connected, list files:

```bash
ls
```

If `flag.txt` is visible, download it:

```bash
get flag.txt
```

File is saved to your current local directory.

---

## 🧠 Takeaways

- FTP servers allowing anonymous login are a **major security risk**.
- Default scripts (`-sC`) during Nmap enumeration often catch these open doors.
- Always check for anonymous access on open FTP ports.

---

## 📌 Summary Table

| Step         | Command                                | Description                          |
|--------------|----------------------------------------|--------------------------------------|
| Port scan    | `nmap -sV -sC -O 10.129.201.243`       | Enumerates services, OS, scripts     |
| Connect FTP  | `ftp 10.129.201.243`                   | Launch FTP client                    |
| Login        | `anonymous` / (no password)            | Use default anonymous creds          |
| List files   | `ls`                                   | View FTP directory contents          |
| Download     | `get flag.txt`                         | Retrieve target file                 |

---

This is a classic low-hanging fruit example—useful for beginners to understand real-world misconfigurations.
