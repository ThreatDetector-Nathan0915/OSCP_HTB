# RDP — empty password on built-in administrator (lab)

Misconfigured **Remote Desktop** can allow interactive logon when the **`Administrator`** account has **no password** (or a trivial password) and policy does not block it. This is **critical** severity on real networks; here it is documented as a controlled lab exercise.

---

## 1. Port scan

**Command:**

```bash
nmap -sV -p- <target-ip>
```

| Flag | Purpose |
|------|---------|
| `-sV` | Service/version on each open port |
| `-p-` | Full TCP range (use top-1000 first on wide scans to save time) |

**Expected:** `3389/tcp open microsoft-rdp` (or equivalent RDP stack string).

Replace `<target-ip>` with the lab address.

---

## 2. `xfreerdp` usage

**Help:**

```bash
xfreerdp --help
```

Common switches for quick tests:

| Switch | Meaning |
|--------|---------|
| `/u:<user>` | Username |
| `/v:<host>` | Target host or IP |
| `/p:<password>` | Password (omit to be prompted) |
| `/cert:ignore` | Skip cert validation (**lab only**; unsafe otherwise) |

---

## 3. Connect with empty password

```bash
xfreerdp /u:administrator /v:10.129.1.13
```

When prompted for a password, **press Enter** if the account truly has no password (lab condition).

**QC:** On modern Windows, empty-password network logons are usually **blocked** by policy—this behavior is **CTF/lab specific**. Document GPO expectations (`MinimumPasswordLength`, `LimitBlankPasswordUse`) for comparison.

---

## 4. Post-access

- Use the remote desktop session to open **`flag.txt`** (or equivalent) from the user profile / Desktop as instructed.
- Do not enable RDP or weaken production hosts to reproduce this.

---

## Summary

| Step | Command | Description |
|------|---------|-------------|
| Discovery | `nmap -sV -p- <target-ip>` | Locate RDP (`3389`) |
| Client help | `xfreerdp --help` | Confirm syntax |
| Logon | `xfreerdp /u:administrator /v:<target-ip>` | Empty password attempt |
| Objective | GUI | Retrieve flag per lab |

---

## Lessons learned

- RDP exposure + weak **`Administrator`** posture = full GUI compromise.
- **`xfreerdp`** is a standard Linux RDP client for testing and operator access.
- In assessments, pair this finding with **local security policy** export and **patch** level evidence.

**Status:** lab completed.
