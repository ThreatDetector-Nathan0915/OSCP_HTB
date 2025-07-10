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

