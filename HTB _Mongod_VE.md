# MongoDB enumeration — notes

MongoDB often listens on **27017/tcp** (default) or on a **high port** if configured non-standard. Confirm with a full port scan, then use **`mongosh`** (or legacy `mongo`) for interactive queries.

---

## 1. Port scan

**Command:**

```bash
nmap -p- -sV <target-ip>
```

| Flag | Purpose |
|------|---------|
| `-p-` | Scan all TCP ports (finds non-default MongoDB binds) |
| `-sV` | Version detection — banner often shows `mongodb` and version |

**QC:** Replace `<target-ip>` with the lab IP. For faster first pass you can use `-p 27017,28017` then widen if needed.

---

## 2. Identify MongoDB in results

Look for lines similar to:

```text
PORT      STATE SERVICE VERSION
27017/tcp open  mongodb MongoDB 4.4.0
```

Note **port**, **version**, and whether **authentication** is mentioned in follow-up probes (e.g. `mongosh` connect attempt, or Nmap `mongodb-*` scripts if allowed).

---

## 3. Install `mongosh` (example — Linux x64)

**Commands:**

```bash
curl -O https://downloads.mongodb.com/compass/mongosh-2.3.2-linux-x64.tgz
tar xfv mongosh-2.3.2-linux-x64.tgz
cd mongosh-2.3.2-linux-x64
./bin/mongosh --version
```

**QC:**

- Download a **mongosh** build that matches your OS/arch (URL and version change over time).
- Prefer **package manager** installs on production systems; tarball is fine for ad hoc lab use.

---

## 4. Connect (no auth / lab)

**Command:**

```bash
./bin/mongosh "mongodb://<target-ip>:<port>"
```

**With authentication (when required):**

```bash
mongosh "mongodb://user:pass@<target-ip>:<port>/?authSource=admin"
```

| URI part | Meaning |
|----------|---------|
| `user:pass` | Credentials |
| `authSource=admin` | DB where user is defined (common default) |

---

## 5. Basic `mongosh` / shell commands

**List databases:**

```javascript
show dbs
```

**Select a database:**

```javascript
use <database-name>
```

**List collections:**

```javascript
show collections
```

**Read documents (limit output in large collections):**

```javascript
db.<collection-name>.find().limit(20).pretty()
```

**QC:** Replace `<database-name>` and `<collection-name>` with values from `show dbs` / `show collections`. Avoid dumping huge collections without `limit()` in client environments.

---

## Security note

Unauthenticated MongoDB on a routable interface is a **critical** misconfiguration. In assessments, document exposure, sample non-sensitive documents as evidence, and recommend **bind address**, **firewall**, and **SCRAM**/TLS configuration per vendor guidance.
