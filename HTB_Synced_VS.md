# 📦 Rsync Enumeration & Exploitation – Full Walkthrough

This note details a host enumeration and exploitation scenario using `rsync`. `rsync` is a file synchronization protocol similar to `ftp`, but with a key advantage: **it only transfers files that have changed**, making it much more efficient for updates or backups. Most Unix/Linux distributions include `rsync` by default.

---

## 🔍 Step 1: Nmap Scan

We start by scanning all ports and service versions:

```bash
nmap -sV -p- 10.129.228.37
```

### 🧾 Result:
Only one port is open:
- **TCP (873)** running `rsync` service.

This is a strong indicator that the host is offering open or misconfigured file shares via `rsync`.

---

## 📂 Step 2: Discover Rsync Shares

To enumerate accessible `rsync` modules (shares), we use the `--list-only` flag:

```bash
rsync --list-only 10.129.228.37::
```

### ✅ Output Example:
```
public          Public rsync share
```

This shows a `public` share is available.

---

## 📁 Step 3: List Contents of a Share

Next, we inspect the contents of the `public` share:

```bash
rsync --list-only 10.129.228.37::public
```

This command displays the files or directories exposed inside the `public` share.

---

## 📥 Step 4: Download Files from Share

If we see something of interest (e.g., `flag.txt`), we can download it using a standard `rsync` pull (without the `--list-only` flag):

```bash
rsync 10.129.228.37::public/flag.txt flag.txt
```

This downloads `flag.txt` into the current working directory.

> ✅ You now have a local copy of the file.

---

## 📌 Summary

| Step             | Command                                           | Purpose                              |
|------------------|---------------------------------------------------|--------------------------------------|
| Scan ports       | `nmap -sV -p- <IP>`                               | Finds open `rsync` port              |
| List shares      | `rsync --list-only <IP>::`                       | Enumerates all available shares      |
| List contents    | `rsync --list-only <IP>::<share>`                | Views share contents                 |
| Download file    | `rsync <IP>::<share>/filename filename`          | Pulls file from target               |

---

## 🧠 Takeaways

- `rsync` is more efficient than `ftp` because it only syncs changes.
- Misconfigured public `rsync` shares can expose sensitive files.
- You don’t need authentication if the share is world-readable.

Let me know if you'd like this exported with command syntax highlighting or zipped as a reusable markdown template.
