# Web and DNS enumeration — reference

WHOIS, DNS resolution flow, record types, enumeration tools, virtual hosts, web scanners, crawling, Google dorks, and **FinalRecon**. Use **authorized targets only**.

---

## WHOIS

WHOIS returns **registrar**, **contacts**, **dates**, and **name servers** for a domain.

**Typical fields:**

```text
Domain Name:        Registered hostname (e.g. example.com)
Registrar:          Company that sold/registered the domain
Registrant Contact: Legal registrant (often redacted under GDPR/RDAP)
Admin / Tech:       Operational contacts (may be privacy-protected)
Created / Expires:  Registration lifecycle timestamps
Name Servers (NS):  Hostnames that delegate DNS for the zone
```

**CLI example:**

```bash
whois example.com
```

Prefer **RDAP** where available (`curl https://rdap.org/domain/example.com`) for structured JSON in automation.

---

## How DNS resolution works (summary)

1. **Stub resolver** on your host checks **local cache** (`/etc/hosts`, OS resolver cache).
2. If missing, the query goes to a **recursive resolver** (ISP, `1.1.1.1`, `8.8.8.8`, etc.).
3. Resolver walks the hierarchy: **root** → **TLD** (`.com`) → **authoritative** nameserver for the zone.
4. The **answer** (A/AAAA/CNAME/…) is cached with a **TTL**.

**`/etc/hosts`** overrides DNS for static mappings on **that machine only**—useful for lab hostnames, dangerous if abused for phishing.

---

## DNS concepts (table)

```text
Concept              Description                                      Example
Domain name          Human-readable name                              www.example.com
IP address           Network endpoint                               192.0.2.1
DNS resolver         Performs recursion/caching for clients           1.1.1.1, 8.8.8.8
Root / TLD / Auth    Hierarchy levels                                 .com TLD → registrar NS
Record types         A, AAAA, CNAME, MX, NS, TXT, SOA, SRV, PTR       See below
```

---

## Common record types

```text
Type   Purpose                              Example snippet
A      IPv4 address                         www IN A 192.0.2.1
AAAA   IPv6 address                         www IN AAAA 2001:db8::1
CNAME  Alias to another name                blog IN CNAME web.example.net.
MX     Mail exchanger                       IN MX 10 mail.example.com.
NS     Delegates zone to nameservers        IN NS ns1.example.com.
TXT    Arbitrary text (SPF, DKIM, verify)   IN TXT "v=spf1 mx -all"
SOA    Zone metadata + serial               Primary NS, contact, timers
SRV    Service location                     _sip._udp SRV 10 5 5060 sip.example.com.
PTR    Reverse IP → hostname                1.0.168.192.in-addr.arpa. PTR host.example.com.
```

---

## DNS enumeration tools (overview)

```text
Tool        Role
dig         Full-featured queries, `@server`, `+trace`, `axfr` when allowed
nslookup    Simple interactive / one-shot queries (Windows/Linux)
host        Short A/AAAA/MX answers
dnsenum     Subdomain brute force, zone transfer attempts, zone walking options
fierce      Linked/recursive subdomain discovery
dnsrecon    Combined DNS checks, CSV/XML output
theHarvester Emails/subdomains from search engines and public sources
```

---

## `dig` usage (cheat sheet)

```text
Command                         Description
dig example.com                 Default A/AAAA lookup path
dig example.com MX              Mail exchangers
dig example.com NS              Delegated nameservers
dig example.com TXT             TXT records (SPF, verification tokens)
dig @1.1.1.1 example.com        Query a specific resolver
dig +trace example.com          Iterative resolution trace
dig -x 192.168.1.1              PTR / reverse (may need `-b` / auth NS)
dig +short example.com          Minimal answer only
dig +noall +answer example.com  Answer section only
dig example.com ANY             Often blocked (RFC 8482); avoid in production scans
```

**Zone transfer (only if authorized / misconfigured):**

```bash
dig axfr example.com @10.129.144.236
```

| Part | Meaning |
|------|---------|
| `axfr` | Asks for a **full zone transfer** |
| `@NS` | Target nameserver IP or name |

**QC:** Most servers return `REFUSED` or `NOTAUTH`—log the response code.

---

## DNS brute force (`dnsenum`)

```bash
dnsenum --enum inlanefreight.com -f /home/kali/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
```

| Flag / arg | Purpose |
|------------|---------|
| `--enum` | Shortcut preset for enumeration-oriented checks |
| `-f` | Wordlist path (adjust to your SecLists install) |
| `-r` *(optional)* | Recursive brute on discovered child domains (noisy) |

**QC:** Add `-r` only when you intend deeper recursion; it multiplies queries.

---

## Virtual host (VHOST) fuzzing (`ffuf`)

When one IP serves many sites, the **`Host`** header selects the vhost:

```bash
ffuf -w /home/kali/SecLists/Discovery/DNS/namelist.txt \
  -u http://inlanefreight.htb:53295 \
  -H "Host: FUZZ.inlanefreight.htb" \
  -fs 116
```

| Flag | Purpose |
|------|---------|
| `-w` | Wordlist for hostname prefix |
| `-u` | Base URL (scheme + IP + port) |
| `-H` | Header template; `FUZZ` is replaced per word |
| `-fs` | **Filter response size** — hides identical “default” pages |

Tune `-fs`, `-fc`, or `-fr` after a baseline **calibration** request.

---

## Web stack fingerprinting

- **Wappalyzer** (browser extension) — quick tech stack hints from HTML/headers.
- **Nikto** — web server misconfiguration / dangerous file checks (noisy).

```bash
nikto -h http://app.inlanefreight.local -Tuning b
```

`-Tuning b` limits Nikto to **“potentially sensitive software”** checks (see `nikto -Help` for tuning sets).

---

## ReconSpider (Scrapy-based crawl)

**Install:**

```bash
pip3 install scrapy
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
unzip ReconSpider.zip
```

**Run:**

```bash
python3 ReconSpider.py http://inlanefreight.com
```

**Output:** `results.json` — review for hidden paths, forms, and linked assets.

**QC:** Prefer a **venv** (`python3 -m venv .venv && source .venv/bin/activate`) before `pip install` on modern Debian/Kali.

---

## Google dorking (operator reference)

```text
Operator        Example                         Notes
site:           site:example.com                Restrict to a site
inurl:          inurl:admin login               Path contains token(s)
filetype:       filetype:pdf budget             Indexed file types
intitle:        intitle:"index of"             HTML title match
intext:         intext:"default password"      Body match
cache:          cache:example.com               Cached copy (availability varies)
```

Combine with `AND` / `OR` / quotes for precision; respect **robots**, **rate limits**, and **legal scope**.

---

## FinalRecon (automated recon)

**Install from Git:**

```bash
git clone https://github.com/thewhiteh4t/FinalRecon.git
cd FinalRecon
chmod +x ./finalrecon.py
./finalrecon.py --help
```

**Dependency install (pick one approach):**

- **System pip (Kali / permissive):**

```bash
sudo python3 -m pip install --break-system-packages -r requirements.txt
```

- **Recommended — virtualenv:**

```bash
sudo apt update
sudo apt install -y python3-venv
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -U pip setuptools wheel "tldextract>=5.3.0"
pip install -r requirements.txt
python finalrecon.py --full --url http://inlanefreight.com
```

**Example modules run:**

```bash
./finalrecon.py --headers --whois --url http://inlanefreight.com
```

**QC:** `--full` is aggressive; confirm bandwidth and **RoE** before use.
