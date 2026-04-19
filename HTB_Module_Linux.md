# Linux prompts, `~/.bashrc`, and command reference

---

## Shell prompt symbols

| Symbol | Meaning |
|--------|---------|
| `#` | **root** (effective UID 0) — destructive commands affect the whole system |
| `$` | Unprivileged user |

Some distributions use **`>`** for `fish` or custom `PS1` strings—always verify with `whoami` / `id` before running privileged commands.

---

## Customize the prompt (`~/.bashrc`)

**Edit:**

```bash
nano ~/.bashrc
```

Changes to **`PS1`** (and `PROMPT_COMMAND`) take effect in new shells, or immediately with:

```bash
source ~/.bashrc
```

**Generator (visual prompt builder):** [bash-prompt-generator.org](https://bash-prompt-generator.org)

---

## Command cheat sheet

### Files and directories

```bash
ls              # List directory
pwd             # Current directory
cd <dir>        # Change directory
mkdir <dir>     # Create directory
rm <file>       # Remove file (irreversible)
rm -r <dir>     # Remove directory tree — double-check path
cp <src> <dest> # Copy
mv <src> <dest> # Move/rename
touch <file>    # Create empty file or update mtime
cat <file>      # Print file (use `less` for large files)
```

### Search

```bash
find <path> -name '<pattern>'   # By name; narrow <path> first (avoid starting at / on huge disks)
grep -R "text" <dir>            # Recursive content search
locate <name>                   # Uses updatedb index — run sudo updatedb periodically
```

**QC:** `find /` from root is slow and can hammer network mounts—start under `/home`, `/etc`, or known app paths.

### Permissions

```bash
chmod +x script.sh     # Execute bit
chmod 755 script.sh    # rwxr-xr-x
chown user:group file  # Ownership (requires root)
```

### System info

```bash
uname -a    # Kernel / arch
hostname    # Node name
df -h       # Filesystem use
free -h     # Memory
uptime      # Load average
top         # Interactive process view
whoami      # Current user
id          # UID/GID and groups
```

### Processes and services

```bash
ps aux | head
kill <pid>                 # SIGTERM single PID
sudo systemctl status ssh  # Example unit status
sudo systemctl restart ssh
```

### Debian/Ubuntu packages

```bash
sudo apt update
sudo apt install <pkg>
sudo apt remove <pkg>
dpkg -l | less
```

### Users (requires root for most)

```bash
sudo adduser <name>
sudo passwd <name>
sudo usermod -aG sudo <name>   # Debian/Ubuntu admin group
```

### Networking

```bash
ip a              # Addresses and interfaces
ping -c3 <host>   # Limited ICMP test
ss -tulnp         # Listening sockets + processes (modern `netstat`)
curl -I <url>     # HTTP headers only
wget <url>        # Download to file
```

---

## Summary

| Item | Role |
|------|------|
| `#` / `$` | Quick privilege reminder (not a substitute for `id`) |
| `~/.bashrc` | Per-user shell init and `PS1` |
| Cheat sheet | Everyday navigation, search, services, and network triage |

Keep this as a **quick reference**; prefer **`man <command>`** and **`tldr <command>`** on hosts where they are installed.
