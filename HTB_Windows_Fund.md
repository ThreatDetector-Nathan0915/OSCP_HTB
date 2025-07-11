# HTB Academy – Windows Fundamentals (Pages 1–3)  
**Complete Command Summary**

---

## 💻 PowerShell and WMI Enumeration

```powershell
Get-WmiObject -Class win32_OperatingSystem | select Version,BuildNumber
```
- Retrieves the Windows version and build number using WMI.

```powershell
Get-WmiObject -Class Win32_Process
```
- Lists all running processes via WMI.

```powershell
Get-WmiObject -Class Win32_Service
```
- Lists all system services and their states.

```powershell
Get-WmiObject -Class Win32_Bios
```
- Fetches BIOS information such as manufacturer, version, and serial number.

```powershell
Get-WmiObject -Class Win32_Service -ComputerName <remote_ip>
```
- Queries services on a remote computer using WMI.

---

## 🌐 RDP (Remote Desktop Protocol)

```bash
xfreerdp /v:<targetIp> /u:htb-student /p:Academy_WinFun!
```
- Connects to a remote Windows desktop using RDP with provided credentials.

Breakdown:
- `/v:` = target IP
- `/u:` = username
- `/p:` = password
- Add `/cert-ignore`, `/drive:` or `/clipboard` for extra functionality like file or clipboard sharing.

---

## 🗂️ File System Structure and Navigation

```cmd
dir c:\ /a
```
- Lists all files and directories at root of C:\ including hidden/system files.

```cmd
tree "c:\Program Files (x86)\VMware"
```
- Displays a tree hierarchy of folders under VMware directory.

```cmd
tree c:\ /f | more
```
- Lists all files and directories in C:\ with paging.

---

### Key Folders
- `C:\Windows` → OS core
- `C:\Program Files` → 64-bit apps
- `C:\Program Files (x86)` → 32-bit apps
- `C:\Users` → User profiles
- `C:\ProgramData` → Shared app data
- `C:\System32`, `SysWOW64` → System binaries and DLLs
- `C:\WinSxS` → Component store
- `C:\Users\<user>\AppData` → Per-user data

---

## 🔐 NTFS Permissions

```cmd
icacls c:\windows
```
- Lists NTFS permissions for `C:\Windows`.

```cmd
icacls c:\Users
```
- Lists NTFS permissions for `C:\Users`.

```cmd
icacls c:\users /grant joe:f
```
- Grants Full Access (`F`) to user `joe` on `C:\Users`.

```cmd
icacls c:\users
```
- Re-checks and displays updated ACL (Access Control List).

```cmd
icacls c:\users /remove joe
```
- Removes all permissions for user `joe`.

### NTFS Permission Flags

| Flag | Meaning               |
|------|------------------------|
| F    | Full Control           |
| M    | Modify                 |
| RX   | Read and Execute       |
| R    | Read-only              |
| W    | Write-only             |
| D    | Delete                 |
| N    | No Access              |
| OI   | Object Inherit         |
| CI   | Container Inherit      |
| IO   | Inherit Only           |
| NP   | Do Not Propagate       |
| I    | Inherited              |

---

## 📡 SMB & Share Permissions

```bash
smbclient -L SERVER_IP -U htb-student
```
- Lists all SMB shares available on the server.

```bash
smbclient '\\SERVER_IP\Company Data' -U htb-student
```
- Connects to `Company Data` SMB share interactively.

```bash
sudo mount -t cifs -o username=htb-student,password=Academy_WinFun! //ipaddoftarget/"Company Data" /home/user/Desktop/
```
- Mounts SMB share as a local folder on your Linux system.

```bash
sudo apt-get install cifs-utils
```
- Installs necessary CIFS utilities to mount SMB shares.

---

## 🪟 Windows GUI Tools

### Computer Management
```bash
compmgmt.msc
```
- View existing shares, open files, sessions.

### Event Viewer
```bash
eventvwr.msc
```
- Check logs like Event ID 5059 (e.g. audit failures).

---

## 📌 Summary

- Use `Get-WmiObject` to query system info.
- `xfreerdp` for RDP login with username/password.
- `tree`, `dir`, and `icacls` for filesystem inspection and permission control.
- `smbclient` and `mount -t cifs` for accessing SMB shares.
- NTFS + Share permissions both matter – most restrictive applies.


# 🔐 NTFS vs. Share Permissions (HTB Windows Fundamentals)

## 🔄 Understanding SMB and NTFS Permission Layers

### 🎯 Target
Target IP: `10.129.201.57`  
RDP Username: `htb-student`  
RDP Password: `Academy_WinFun!`

---

## 🔗 What is SMB?
**SMB (Server Message Block)** is the Windows protocol used to share resources like:
- Files
- Folders
- Printers

---

## 🔧 Share Permissions (via GUI)
| Permission    | Description                                                                 |
|---------------|-----------------------------------------------------------------------------|
| Full Control  | Can change permissions and perform all actions                              |
| Change        | Read, edit, delete, add files/folders                                        |
| Read          | View content only                                                           |

---

## 🔒 NTFS Permissions
| Permission         | Description                                                            |
|--------------------|------------------------------------------------------------------------|
| Full Control       | Add/edit/delete files & folders, change NTFS perms                     |
| Modify             | View and modify files/folders                                          |
| Read & Execute     | Read file content and execute                                          |
| List folder        | View contents of folder                                                |
| Read               | Read contents of files                                                 |
| Write              | Add new files, edit existing                                           |
| Special Permissions| Fine-grained permissions                                               |

---

## 🧱 NTFS Inheritance
By default, NTFS permissions are **inherited** from parent directories (e.g., `C:\`).

Can be disabled via:
- File > Properties > Security > Advanced > Disable inheritance

---

## 💻 Creating the Shared Folder
1. Create new folder on Desktop: `Company Data`
2. Right-click → Properties → Sharing → Advanced Sharing
3. Check **"Share this folder"**
4. Set name: `Company Data`
5. Set permission for `Everyone` to `Read`

---

## 📡 Accessing Shares Remotely (from Pwnbox)
### 🔍 Listing shares:
```bash
smbclient -L 10.129.201.57 -U htb-student
```

### 🔗 Connect to share:
```bash
smbclient '\\\\10.129.201.57\\Company Data' -U htb-student
```

---

## 🧱 What blocks access? Windows Defender Firewall!
Disable firewall or allow inbound SMB via:
- Control Panel → Windows Defender Firewall → Advanced Settings
- Enable: **File and Printer Sharing (SMB-In)** for Public profile

---

## 📂 Mounting SMB Share to Linux
```bash
sudo mount -t cifs -o username=htb-student,password=Academy_WinFun! //10.129.201.57/"Company Data" /home/kali/Desktop/
```

If mount fails:
```bash
sudo apt-get install cifs-utils
```

---

## 🧰 Auditing Shares (from within Windows)

### View shares
```cmd
net share
```

### Sample Output
```
Share name   Resource                          Remark
---------------------------------------------------------------
C$           C:\                               Default share
IPC$                                           Remote IPC
ADMIN$       C:\WINDOWS                        Remote Admin
Company Data C:\Users\htb-student\Desktop\Company Data
```

---

## 📊 View Logs of Share Access

### Open Event Viewer:
```
eventvwr.msc
```

Navigate to:
```
Windows Logs > Security > Event ID 5059
```

---

# 🧠 Windows Services & Processes (Page 5)

## 🔎 Check Running Services (PowerShell)
```powershell
Get-Service | ? {$_.Status -eq "Running"} | Select -First 5 | Format-List
```

Sample Output:
```
Name                : AdobeARMservice
DisplayName         : Adobe Acrobat Update Service
Status              : Running
...
```

---

## 🧵 Critical Windows Processes (do not terminate!)
| Process         | Description                                           |
|------------------|-------------------------------------------------------|
| smss.exe         | Session Manager                                       |
| csrss.exe        | Client Server Runtime Subsystem                       |
| lsass.exe        | Local Security Authority Subsystem Service            |
| services.exe     | Controls service startup/stop                         |
| winlogon.exe     | Handles secure login and user profile loading         |
| svchost.exe      | Hosts services from DLLs (RPCSS, DCOM, etc.)          |

---

## 🧰 Using Sysinternals Tools (no download needed)

Run tool from remote:
```cmd
\\live.sysinternals.com\tools\procdump.exe -accepteula
```

Useful tools:
- `procdump` — Dumps process memory
- `Process Explorer` — Task Manager++ (DLLs, handles, parent/child)
- `Process Monitor` — Tracks registry, file, and network activity
- `TCPView` — Shows live open TCP/UDP connections
- `PsExec` — Remote execution over SMB

---

## 🛠️ Task Manager Tabs Breakdown

| Tab             | Info Provided |
|-----------------|----------------|
| Processes       | CPU, RAM, Disk, Network |
| Performance     | CPU/mem usage graphs |
| App History     | App-specific stats |
| Startup         | Boot auto-start apps |
| Users           | Logged in user processes |
| Details         | PID, username, CPU/mem |
| Services        | List of services |

## ✅ Summary:
- NTFS and Share permissions combine!
- Use `smbclient` or mount SMB shares to test access
- Use `net share`, `eventvwr.msc`, and `services.msc` for auditing
- Sysinternals tools are powerful for Windows internals

# HTB Academy – Windows Fundamentals (Pages 6-8)

## 🔐 Windows Service Permissions

Windows services are long-running processes that are essential for OS operation. Misconfigurations in service permissions can lead to privilege escalation or persistent malware execution.

### 🔧 Examining Services via `services.msc`
This GUI utility allows you to inspect:

- Service name (used in CLI tools)
- Executable path (`Path to executable`)
- Logon options (`LocalSystem`, custom service accounts)
- Recovery options (e.g., run a program on failure)

### 🔍 Examining Services with `sc`

```cmd
sc qc wuauserv
```
> Queries config for the `Windows Update` service.

```cmd
sc //hostname query ServiceName
```
> Queries a service on a remote host.

```cmd
sc stop wuauserv
```
> Attempts to stop service (requires elevated privileges).

```cmd
sc config wuauserv binPath= C:\Winbows\Perfectlylegitprogram.exe
```
> Replaces service binary path.

```cmd
sc sdshow wuauserv
```
> Dumps the security descriptor in SDDL format.

### 🔍 PowerShell: View Service Permissions via Registry

```powershell
Get-Acl -Path HKLM:\System\CurrentControlSet\Services\wuauserv | Format-List
```

This shows:

- Owner
- Group
- Access control entries (ACE)
- SDDL format permissions

---

## 🧑‍💻 Windows Sessions

### ✅ Interactive (Local or RDP Logons)
- User authenticates via keyboard login or RDP
- Initiates an interactive session

### ❌ Non-Interactive
Run services & tasks *without* login. These accounts include:

| Account | Description |
|--------|-------------|
| `LocalSystem` | NT AUTHORITY\SYSTEM, highest privilege |
| `LocalService` | Limited local privilege |
| `NetworkService` | Local + network authentication |

---

## 🖱️ Interacting with Windows

### 🪟 GUI (Graphical User Interface)
- Introduced to ease usability
- Used by sysadmins for managing AD, IIS, DBs, etc.

### 🔌 RDP (Remote Desktop Protocol)
- GUI over network on TCP 3389
- Admins use this to manage systems remotely

---

## 🖥️ Windows Command Line (CMD)

### Run CMD

```cmd
help
```

Lists available commands. Example:

```cmd
help schtasks
```

```cmd
ipconfig /?
```

Displays usage & arguments of command.

---

## 💪 Windows PowerShell

PowerShell is built on .NET and includes advanced scripting capabilities and cmdlets.

### 📜 Cmdlets: Verb-Noun pattern

```powershell
Get-ChildItem -Recurse
Get-ChildItem -Path C:\Users\Administrator\Downloads -Recurse
```

### 🔁 Aliases

```powershell
Get-Alias
Get-Alias -Name "cd"
New-Alias -Name "Show-Files" Get-ChildItem
```

### 📚 Help System

```powershell
help
Get-Help Get-AppPackage
Get-Help Get-AppPackage -Online
Update-Help
```

---

## ⚙️ Running PowerShell Scripts

```powershell
.\PowerView.ps1; Get-LocalGroup | fl
Import-Module .\PowerView.ps1
Get-Module | select Name,ExportedCommands | fl
```

---

## 🔒 Execution Policies

| Policy | Description |
|--------|-------------|
| AllSigned | Only run signed scripts |
| Bypass | No warnings |
| RemoteSigned | Requires signature for remote scripts |
| Restricted | No scripts allowed (default) |
| Unrestricted | All scripts allowed with warnings |

### View Execution Policy

```powershell
Get-ExecutionPolicy -List
```

### Bypass for Session

```powershell
Set-ExecutionPolicy Bypass -Scope Process
```

Confirm policy change:

```plaintext
[Y] Yes  [A] Yes to All  [N] No
```

---

## ✅ Module Quiz Answers

- ❓ **Alias for ipconfig.exe**: `ifconfig`
- ❓ **Execution Policy (LocalMachine)**: `Unrestricted`
# Windows Fundamentals – Part 4: Windows Management Instrumentation (WMI)

## WMI Overview

Windows Management Instrumentation (WMI) is a powerful subsystem built into Windows designed for system management, monitoring, and scripting. It's an interface for accessing management information in an enterprise environment and is widely used by both sysadmins and adversaries.

### Core Components of WMI

| Component Name       | Description                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| WMI Service          | Runs at boot and intermediates between providers, repository, and tools     |
| Managed Objects      | Logical or physical components manageable by WMI                            |
| WMI Providers        | Objects monitoring specific event/data sources                              |
| Classes              | Used by providers to pass data to WMI service                               |
| Methods              | Attached to classes to perform actions (start/stop/etc.)                    |
| WMI Repository       | Stores all static WMI data                                                  |
| CIM Object Manager   | Handles data requests and returns to application                            |
| WMI API              | Enables application access to WMI                                           |
| WMI Consumer         | Sends queries to objects via the CIM Object Manager                         |

---

## Common WMI Use Cases

- Monitoring local/remote system status
- Setting and changing user/group permissions
- Configuring security on remote systems
- System property enumeration/modification
- Code execution (locally/remotely)
- Process scheduling
- Logging setup and management

---

## WMIC (WMI Command-Line Interface)

### Launching WMIC Shell or Executing One-Off Commands

```bash
# Open WMIC shell
C:\htb> wmic

# Run one-off command to get computer name
C:\htb> wmic computersystem get name
```

### Example: Listing OS Information

```bash
C:\htb> wmic os list brief

BuildNumber  Organization  RegisteredUser  SerialNumber             SystemDirectory      Version
19041                      Owner           00123-00123-00123-AAOEM  C:\Windows\system32  10.0.19041
```

### WMIC Help

```bash
C:\htb> wmic /?
```

This shows available global switches such as:

- `/NAMESPACE`, `/NODE`, `/ROLE`, `/USER`, `/PASSWORD`
- `/OUTPUT`, `/APPEND`, `/FAILFAST`, `/AUTHORITY`

---

## PowerShell and WMI

### Get-WmiObject

The `Get-WmiObject` cmdlet is used to retrieve instances of WMI classes.

```powershell
PS C:\htb> Get-WmiObject -Class Win32_OperatingSystem | select SystemDirectory,BuildNumber,SerialNumber,Version | ft
```

**Output:**

```
SystemDirectory     BuildNumber SerialNumber            Version
---------------     ----------- ------------            -------
C:\Windows\system32 19041       00123-00123-00123-AAOEM 10.0.19041
```

---

### Invoke-WmiMethod

Used to call WMI class methods (e.g., rename a file).

```powershell
PS C:\htb> Invoke-WmiMethod -Path "CIM_DataFile.Name='C:\users\public\spns.csv'" -Name Rename -ArgumentList "C:\Users\Public\kerberoasted_users.csv"
```

**Output:**

```
ReturnValue : 0
```

---

## Practical Application – HTB Academy Prompt

Target: `10.129.201.57`  
User: `htb-student`  
Password: `Academy_WinFun!`

### Objective: Get Serial Number with WMI

```powershell
Get-WmiObject -Class Win32_BIOS | Select-Object SerialNumber
```

Use this to answer:  
🟢 “Use WMI to find the serial number of the system.”

---

## Summary

- WMI is a Windows-native system management interface.
- WMIC (deprecated) and PowerShell (`Get-WmiObject`, `Invoke-WmiMethod`) are core tools.
- Can be used to interact with hardware info, service management, code execution.
- Extensively used in red/blue team ops, DFIR, and system administration.
