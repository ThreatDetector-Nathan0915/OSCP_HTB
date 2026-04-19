# SEC555 — Cowrie honeypot and Wazuh enrichment (lab notes)

Structured notes for deploying **Cowrie** (SSH honeypot), capturing sessions, and optional **AbuseIPDB** enrichment in **Wazuh**. Replace all IPs, API keys, and SSH public keys with your own lab values.

---

## 1. Honeypot configuration (Cowrie)

### `userdb.txt` (example)

Define credentials the honeypot will accept (intentionally weak for capture-only labs):

```text
root:x:rootpasswrd
```

| Field | Meaning |
|-------|---------|
| `root` | Login name presented to attackers |
| `x` | Password field placeholder in Cowrie userdb format |
| `rootpasswrd` | Cleartext password Cowrie will accept |

### `cowrie.cfg`

- Set **SSH banner / software version** strings to mimic desired targets.
- Use your editor’s search (**Ctrl+W** in `nano`) to jump to `ssh_version`, `hostname`, and listen **port** (often **2222** mapped from Docker).

**QC:** Never reuse production passwords; isolate the honeypot on a lab VLAN.

---

## 2. Enumeration from Kali

**Service scan against the honeypot IP:**

```bash
nmap -sV -sC 192.168.29.133
```

| Flag | Purpose |
|------|---------|
| `-sV` | Version detection |
| `-sC` | Default scripts (often shows SSH banner quirks) |

---

## 3. Brute-force SSH (lab exercise)

**Hydra** against the honeypot SSH port (here **2222**):

```bash
hydra -l root -P /home/kali/SecLists/Passwords/Common-Credentials/xato-net-10-million-passwords.txt ssh://192.168.29.133:2222
```

| Flag | Purpose |
|------|---------|
| `-l root` | Single username |
| `-P` | Password wordlist path |
| `ssh://IP:PORT` | Target service URL form |

**QC:** Use a **truncated** wordlist in class demos; full multi-million lists are noisy and slow.

---

## 4. SSH login (after password found)

```bash
ssh root@192.168.29.133 -p 2222
```

---

## 5. Attacker SSH key pinning (lab scenario)

The exercise may require **replacing** `authorized_keys` so only your analyst key is trusted:

```bash
rm -rf ~/.ssh
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Append **your** public key (one line, `ssh-rsa` / `ssh-ed25519` …):

```bash
echo "ssh-ed25519 AAAA...your-key-here... analyst@lab" >> ~/.ssh/authorized_keys
```

**QC:** The course material originally embedded a long RSA key—**rotate** any key that was ever pasted into coursework. Never paste production private keys into notes.

Clear shell history if the rubric requires it:

```bash
history -c
exit
```

---

## 6. Replay captured TTY logs (Cowrie)

Recorded sessions live under Cowrie’s data directory, for example:

```bash
cd /home/cowrie/cowrie/var/lib/cowrie/tty
ls -lt
```

**Play back** a specific session log:

```bash
python3 /home/cowrie/cowrie/bin/playlog ./<SESSION_LOG_FILENAME>
```

Replace `<SESSION_LOG_FILENAME>` with the artifact Cowrie created (long hex name).

---

## 7. Wazuh — AbuseIPDB integration (outline)

1. Create an **AbuseIPDB** account and API key (free tier for labs).
2. Install helper script with correct ownership:

```bash
sudo chmod 755 /var/ossec/integrations/custom-abuseipdb.py
sudo chown root:wazuh /var/ossec/integrations/custom-abuseipdb.py
```

3. Edit **`/var/ossec/etc/ossec.conf`** and add an `<integration>` block (use **environment variables** or a **restricted** config template—avoid committing real API keys to git):

```xml
<!-- AbuseIPDB integration (example — use YOUR key) -->
<integration>
  <name>custom-abuseipdb.py</name>
  <hook_url>https://api.abuseipdb.com/api/v2/check</hook_url>
  <api_key>YOUR_ABUSEIPDB_API_KEY</api_key>
  <level>10</level>
  <rule_id>100002</rule_id>
  <alert_format>json</alert_format>
</integration>
```

4. Tail integration logs:

```bash
sudo tail -f /var/ossec/logs/integrations.log
```

5. Check manager / stack health:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-dashboard
sudo systemctl status wazuh-indexer
```

6. List custom integrations and sample rules:

```bash
ls -l /var/ossec/integrations
ls /var/ossec/logs/alerts/
ls /var/ossec/ruleset/rules/ | grep vsftpd
```

---

## 8. Optional — SSH server on Linux test host

```bash
sudo apt-get update
sudo apt-get install -y openssh-server
sudo systemctl start ssh
```

Use only on **isolated** lab networks.

---

## QC checklist

- [ ] All IPs and API keys are **placeholders** or redacted for sharing.
- [ ] Cowrie listens on a **non-standard** port and is **firewalled** from production.
- [ ] Wazuh integration tested with a **known benign** IP before blocking automation runs.
