# Nmap, FTP, SMB, and Service Enumeration Cheat Sheet — HTB Enumeration Notes

---

## Target: 10.129.68.125

### Nmap full service scan

```bash
sudo nmap -sC -sV 10.129.68.125
```

| Flag | Purpose |
|------|---------|
| `sudo` | Often used so **SYN** scans and some payloads work consistently (policy-dependent) |
| `-sC` | **Default NSE** script set—safe discovery scripts for common ports |
| `-sV` | **Version** detection and service fingerprinting |

**Findings:**

| Port     | State | Service     | Version                            |
|----------|-------|-------------|------------------------------------|
| 22/tcp   | open  | ssh         | OpenSSH 7.6p1 Ubuntu 4ubuntu0.7    |
| 80/tcp   | open  | http        | Apache 2.4.29 (Ubuntu)             |
| 110/tcp  | open  | pop3        | Dovecot pop3d                      |
| 139/tcp  | open  | netbios-ssn | Samba smbd 3.X - 4.X               |
| 143/tcp  | open  | imap        | Dovecot imapd (Ubuntu)             |
| 445/tcp  | open  | netbios-ssn | Samba smbd 4.7.6-Ubuntu            |
| 31337/tcp| open  | ftp         | ProFTPD                            |

---

## FLAG FOUND

```bash
ftp 10.129.68.125 31337
```

| Syntax | Note |
|--------|------|
| `ftp HOST PORT` | Classic FTP client: connect to **non-default** port **31337** |

**Output:**
```text
220 HTB{pr0F7pDv3r510nb4nn3r}
```

 **Flag:** `HTB{pr0F7pDv3r510nb4nn3r}` — Found directly in the FTP banner message!

---

## FTP Service (Port 31337)

```bash
ftp 10.129.68.125 31337
```

- Tried anonymous login:
  - `USER: anonymous`
  - `PASS: (blank)`
  -  Result: `530 Login incorrect`

- Attempted:
```bash
  ftp> ls
```
   Result: Must login first

- **Important Note:** Even without login, banner shows the flag `HTB{...}`

---

## SMB Enumeration

```bash
smbclient -L 10.129.68.125
```

**Output:**

```text
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
IPC$            IPC       IPC Service (nix-nmap-default server)

Workgroup: WORKGROUP
```

Tried connecting with:

```bash
smbclient \\\\10.129.68.125\\print\$ -U guest
```

 Result: `NT_STATUS_ACCESS_DENIED`

Also tried:

```bash
smbclient \\\\10.129.68.125\\print\$ -U "" -N
smbclient \\\\10.129.68.125\\print\$ -N
```

 Same result: `NT_STATUS_ACCESS_DENIED`

---

## Extra Nmap Knowledge Dump

### TTL Based OS Detection

- Based on `TTL` values in ping replies:
  - **Windows** TTL ~128
  - **Linux** TTL ~64

Reference: https://ostechnix.com/identify-operating-system-ttl-ping/

---

## Nmap Useful Scanning Techniques

### General Ping Scan

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping
```

**Flags explained:**

- `-sn`: Ping scan only (no port scan)
- `-PE`: ICMP Echo request ping
- `--packet-trace`: Show raw packets sent/received
- `--disable-arp-ping`: Skip ARP ping (useful for non-local scans)
- `-oA host`: Output in all formats (XML, grepable, normal)

---

### Version + Vuln Scan

```bash
nmap -p 80 -sV --script vuln 10.129.2.28
```

Finds services and scans for known vulnerabilities on port 80.

---

### Aggressive OS + App Detection

```bash
nmap -p 80 -A 10.129.2.28
```

Does:
- OS Detection
- Service detection
- Traceroute
- Default NSE scripts

---

### UDP Scan

```bash
nmap -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason 10.129.2.28
nmap -sU -Pn -n --disable-arp-ping --packet-trace -p 138 --reason 10.129.2.28
```

**Flags:**

- `-sU`: UDP scan
- `-Pn`: Skip ICMP ping
- `-n`: Skip DNS resolution
- `--disable-arp-ping`: Avoid local network ARP
- `--packet-trace`: Show packet-level info
- `--reason`: Show why Nmap thinks a port is in a specific state

---

### Scan All Ports and Save Output

```bash
nmap 10.129.2.28 -p- -oA target
```

- `-p-`: Scan all 65535 ports
- `-oA target`: Save output in all formats

---

### Set Scan Timing + Verbosity

```bash
nmap --stats-every=5s -v -T4 10.129.2.28
```

- `--stats-every=5s`: Show status every 5 seconds
- `-v` or `-vv`: Increase verbosity
- `-T0` to `-T5`: Set scan speed
  - `T0`: Paranoid
  - `T1`: Sneaky
  - `T2`: Polite
  - `T3`: Normal
  - `T4`: Aggressive
  - `T5`: Insane

---

## NSE Script Categories

Can be run via:
```bash
sudo nmap <target> --script <script-name>,<script-name>
```

Or whole categories:
```bash
--script <category>
```

Categories include:

- `auth` – Credential checks
- `broadcast` – LAN host discovery
- `brute` – Brute force credentials
- `default` – Used with `-sC`
- `discovery` – Service discovery
- `dos` – Denial of service checks
- `exploit` – Known exploits
- `external` – Use external sources
- `fuzzer` – Malformed input testing
- `intrusive` – May crash or affect services
- `malware` – Malware detection
- `safe` – Non-destructive scripts
- `version` – Service version info
- `vuln` – Known vulnerability checks

---

## Final Notes

- ProFTPD on 31337 had the flag right in the banner — perfect example of why banner grabbing matters!
- Always try FTP, HTTP, SMB login attempts (anon, guest, blank) early in enum
- TTL analysis + UDP + NSE scripts = powerful network discovery
- Save your output with `-oA` so you can grep and parse later!


Scanning Options	Description
10.129.2.28	Scans the specified target.
-p 80	Scans only the specified ports.
-sS	Performs SYN scan on specified ports.
-Pn	Disables ICMP Echo requests.
-n	Disables DNS resolution.
--disable-arp-ping	Disables ARP ping.
--packet-trace	Shows all packets sent and received.
-D RND:5	Generates five random IP addresses that indicates the source IP the connection comes from.
# Nmap Command Breakdown — HTB Recon Cheat Sheet (Super Verbose)

This section breaks down **every single `nmap` command used**, with full explanations of each flag and output summaries. Perfect for review or reuse!

---

## Basic Service and Version Detection

```bash
nmap -sC -sV 10.129.68.125
```

- `-sC`: Runs default Nmap scripts (equivalent to `--script=default`)
- `-sV`: Performs service version detection
-  **Use this combo as a first scan** to get info quickly on open ports, banner versions, and common service misconfigurations.

### Output Summary:
Found ports:
- `22/tcp` OpenSSH
- `80/tcp` Apache
- `110/tcp` POP3
- `139/tcp` NetBIOS Samba
- `143/tcp` IMAP
- `445/tcp` NetBIOS Samba
- `31337/tcp` ProFTPD → Banner leak with flag!

---

## HTTP Header Script on a Suspicious Port

```bash
nmap -p 10001 --script http-headers 10.129.2.80
```

- `-p 10001`: Target port 10001
- `--script http-headers`: Run the NSE script to retrieve HTTP response headers

### Output Summary:
- Detected service on `10001/tcp`
- Misidentified as `scp-config` (not actual HTTP)

---

## OS Detection with Performance Tuning

```bash
sudo nmap -O -p 22,80,10001 --osscan-guess --max-retries 1 --min-parallelism 1 --max-rtt-timeout 100ms 10.129.2.80
```

- `-O`: Enable OS detection
- `--osscan-guess`: Make a best-guess attempt even with incomplete data
- `--max-retries 1`: Limit packet retries for faster scan
- `--min-parallelism 1`: Only one probe at a time — reduce stealth
- `--max-rtt-timeout 100ms`: Cap timeout for response wait time

### Output Summary:
- OS Detected: Linux 4.15 – 5.19
- Type: General Purpose

---

## Version Fingerprinting with OS Guess

```bash
sudo nmap -O -sT -p 22,80,10001 --version-all 10.129.2.80
```

- `-O`: OS detection
- `-sT`: TCP connect scan (used when SYN scan isn’t possible)
- `--version-all`: Probe for **all available service version info**

### Output Summary:
- Same ports open as before
- Better OS fingerprint: MikroTik RouterOS 7.X with Linux 5.6.3

---

## Full Aggressive Scan

```bash
sudo nmap -A -p 22,80,10001 10.129.2.80
```

- `-A`: Aggressive scan — includes:
  - OS detection (`-O`)
  - Version detection (`-sV`)
  - Script scan (`-sC`)
  - Traceroute

### Output Summary:
- `22`: OpenSSH 7.6p1
- `80`: Apache 2.4.29 (Ubuntu)
- `10001`: ProFTPD (non-standard port)

---

## DNS Script Scan on UDP

```bash
sudo nmap -sU -p 53 --script=dns-nsid --max-retries 1 --min-rate 1 --max-rtt-timeout 100ms 10.129.67.199
```

- `-sU`: UDP scan mode
- `-p 53`: DNS port
- `--script=dns-nsid`: NSE script to query `BIND` version (reveals flags or leaks)
- `--max-retries 1`: One retry max
- `--min-rate 1`: Send at least 1 packet/sec
- `--max-rtt-timeout 100ms`: Tighten timeout for fast failure

### Output Summary:
- UDP port 53 open
- `bind.version`: **Leaked flag!** `HTB{GoTtgUnyze9Psw4vGjcuMpHRp}`

---

## Summary Table

| Command | Purpose | Key Result |
|--------|---------|-------------|
| `nmap -sC -sV 10.129.68.125` | Default service/script scan | Found FTP flag banner |
| `nmap -p 10001 --script http-headers 10.129.2.80` | Test for HTTP headers | Confirmed odd service |
| `nmap -O --osscan-guess ...` | OS detection w/ low noise | Identified Linux kernel |
| `nmap -O -sT --version-all ...` | Full fingerprinting | MikroTik RouterOS ID |
| `nmap -A ...` | Aggressive everything | Confirmed all details |
| `nmap -sU -p 53 --script=dns-nsid ...` | DNS leak check | Retrieved flag via bind.version |

---

**Suggested workflow:** chain these scans during assessments or labs:
1. `-sC -sV`
2. `-A -p <ports>`
3. `--script vuln` or `--script <specific>`
4. `-sU` for UDP + DNS/NTP weirdness
5. Output to files with `-oA <name>` for full coverage.

That sequence usually yields enough service data to proceed to exploitation or reporting.

# Ultra-Verbose Nmap Command Breakdown (HTB Target Recon)

This is a highly detailed breakdown of each `nmap` command executed during recon on HTB target `10.129.2.47` and related hosts. Each section includes:

-  Purpose
-  Flag breakdown
-  Example output summary
-  Analysis notes

---

## TCP Port Scan + Version Detection (Common Services)

```bash
sudo nmap -sS -sV -p 22,80,443,53,10001 --version-all --max-retries 1 --min-rate 1 --max-rtt-timeout 100ms 10.129.2.47
```

### Explanation:
- `-sS`: SYN scan (stealthy and fast)
- `-sV`: Version detection
- `-p`: Scans specific ports
- `--version-all`: Perform aggressive version detection using all probes
- `--max-retries 1`: Max 1 retry for probes
- `--min-rate 1`: Minimum 1 probe/sec
- `--max-rtt-timeout 100ms`: Reduce waiting time per response (aggressive)

### Summary:
- `22/tcp`: OpenSSH 7.6p1 Ubuntu
- `80/tcp`: Apache 2.4.29 (Ubuntu)
- `53/tcp`, `443/tcp`: Filtered
- `10001/tcp`: Closed

### Notes:
Useful for low-noise scans against firewalled targets where false positives are acceptable.

---

## FTP Probe

```bash
sudo nmap -sS -sV -p 21 --version-all --max-retries 1 --min-rate 1 --max-rtt-timeout 100ms 10.129.2.47
```

### Explanation:
- Focused probe to check for FTP
- Found `21/tcp` closed

---

## Extended Service Set (FTP, SSH, Web, SMB, NFS, Rsync)

```bash
sudo nmap -sS -sV -p 20,21,22,69,80,443,139,445,2049,873,8443 --version-all --max-retries 1 --min-rate 1 --max-rtt-timeout 100ms 10.129.2.47
```

### Summary:
- `22/tcp` open: SSH
- `80/tcp` open: Apache HTTP
- Many others filtered or closed

### Notes:
Used to hit multiple commonly exposed services on CTFs.

---

## Add Suspicious Port `10001` to Full Set

```bash
sudo nmap -sS -sV -p 20,21,22,69,80,443,139,445,2049,873,8443,10001 --version-all --max-retries 1 --min-rate 1 --max-rtt-timeout 100ms 10.129.2.47
```

### Summary:
- Same as above, confirms `10001/tcp` is closed

---

## Full TCP Scan on All 65535 Ports

```bash
sudo nmap -sS -sV -p- --version-all --max-retries 1 --min-rate 1 --max-rtt-timeout 100ms 10.129.2.47
```

### Explanation:
- `-p-`: All 65535 TCP ports
- Full service version fingerprinting

### Output:
- `22/tcp` and `80/tcp` open
- 63,572 ports closed (reset)
- 1,961 filtered (no response)

### Warnings:
- Aggressive timing + retries = retransmission cap hits
- Scan slowed but still completed

---

## UDP DNS NSID Flag Leak

```bash
sudo nmap -sU -p 53 --script=dns-nsid --max-retries 1 --min-rate 1 --max-rtt-timeout 100ms 10.129.67.199
```

### Explanation:
- `-sU`: UDP scan
- `--script=dns-nsid`: Leak DNS software version and sometimes flags!
- Aggressive timing for stealth

### Result:
```text
| dns-nsid: 
|_  bind.version: HTB{GoTtgUnyze9Psw4vGjcuMpHRp}
```

### Flag recovered!

---

## Failing Script Attempt

```bash
sudo nmap -sU -p 53 --script=dns-version --max-retries 1 --min-rate 1 --max-rtt-timeout 100ms 10.129.67.199
```

### Error:
```text
'dns-version' did not match a category, filename, or directory
```

### Lesson:
Always check `ls /usr/share/nmap/scripts/` for valid script names.

---

## Decoy + Aggressive Scan

```bash
sudo nmap -A -p- -D RND:5 --max-retries 1 --min-rate 1 --max-rtt-timeout 100ms 10.129.2.47
```

### Explanation:
- `-A`: Aggressive — version, OS, traceroute
- `-p-`: All ports
- `-D RND:5`: Random decoys for obfuscation

### Error:
```text
Unknown address family 0 in build_packet.
QUITTING!
```

### Notes:
Decoy scans + aggressive timeout may trigger low-level packet bugs or issues with Nmap stack.

---

## Final Tips

-  For stability, use higher `--max-retries` and remove `--min-rate`/`--max-rtt-timeout` when target is stable
-  Use `--script=default,vuln,safe` when ready to escalate
-  Log results: `-oA recon_output` to save `.nmap`, `.xml`, `.gnmap`

---

## Final Recap Table

| Command | Summary |
|--------|---------|
| `-sS -sV -p-` | Full port TCP + version |
| `--version-all` | Force full fingerprinting |
| `--max-retries 1` | Reduce retry noise |
| `--min-rate` | Fast scans (risky on noisy networks) |
| `-sU` | UDP port scan (e.g., DNS flags) |
| `--script=dns-nsid` | Bind version leak and flag discovery |
| `-D RND:5` | Decoys for stealth scans |
| `-p <ports>` | Target specific services (FTP, SSH, HTTP, etc.) |

Together, these patterns support **flag-style challenges**, **port state reasoning** (open / filtered / closed), and **evasion on lightly defended perimeters**.

# Final module

The closing exercise uses **Nmap** against a target protected by **IDS/IPS**-style filtering. Use **slower, lower-rate TCP scanning** and deliberate **source-port** choices where the lab allows, then confirm services with **banner grabbing** (e.g. `ncat`/`nc`) when full version detection is too noisy.

```bash
sudo nmap -sS -sV -Pn -n --source-port 53 --scan-delay 200ms --max-rate 30 --version-intensity 1 10.129.73.113
sudo ncat -nv --source-port 53 10.129.73.113 50000
```
