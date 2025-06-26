# 🧠 Redis Enumeration & Exploitation – Quick Win Walkthrough

This was a short but interesting CTF-style challenge involving a misconfigured Redis server that allowed unauthenticated access.

---

## 🔍 1. Nmap Scan

Initial enumeration was done using Nmap. It took a while due to slow response times, but it eventually revealed:

- Open port running **Redis** (typically port 6379)
- No authentication required for access

```bash
nmap -sV -p- <target-ip>
```

---

## 📡 2. Connecting to Redis

Used the `redis-cli` tool to connect directly to the Redis service:

```bash
redis-cli -h <target-ip>
```

Once connected, ran the `info` command to dump Redis configuration and server details:

```bash
info
```

This gives useful metadata like:
- Redis version
- OS and architecture
- Uptime, connected clients, etc.

---

## 🗝️ 3. Enumerating Keys

Listed all keys in the Redis database using:

```bash
keys *
```

This dumped out all keys present — in this case, there were 5 keys total.

---

## 📖 4. Reading Key Values

To view the value of a specific key (like `flag`), used:

```bash
get <keyname>
```

Example:
```bash
get flag
```

This output the **flag hash** or content, which completed the box.

---

## ✅ Summary

| Step            | Command                        | Description                            |
|-----------------|--------------------------------|----------------------------------------|
| Nmap Scan       | `nmap -sV -p- <target-ip>`     | Identify open Redis port               |
| Connect Redis   | `redis-cli -h <target-ip>`     | Auth-less connection to Redis server   |
| Server Info     | `info`                         | View Redis config and metadata         |
| List Keys       | `keys *`                       | See all Redis DB keys                  |
| Read Value      | `get <key>`                    | Dump value of a specific key (like flag) |

---

## 🎯 Lessons Learned

- Redis often runs without auth in misconfigured boxes.
- `redis-cli` makes access and interaction very easy.
- Always check `keys *` and `get <key>` — quick wins possible.

**Short box, fast win. 💥**
