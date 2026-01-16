# Whois
Great tool for finding out owner and age of a Domain - following whois record information explination.
```bash
Domain Name: The domain name itself (e.g., example.com)
Registrar: The company where the domain was registered (e.g., GoDaddy, Namecheap)
Registrant Contact: The person or organization that registered the domain.
Administrative Contact: The person responsible for managing the domain.
Technical Contact: The person handling technical issues related to the domain.
Creation and Expiration Dates: When the domain was registered and when it's set to expire.
Name Servers: Servers that translate the domain name into an IP address.
```
# DNS General
**How is works**
```bash
Your Computer Asks for Directions (DNS Query): When you enter the domain name, your computer first checks its memory (cache) to see if it remembers the IP address from a previous visit. If not, it reaches out to a DNS resolver, usually provided by your internet service provider (ISP).

The DNS Resolver Checks its Map (Recursive Lookup): The resolver also has a cache, and if it doesn't find the IP address there, it starts a journey through the DNS hierarchy. It begins by asking a root name server, which is like the librarian of the internet.

Root Name Server Points the Way: The root server doesn't know the exact address but knows who does – the Top-Level Domain (TLD) name server responsible for the domain's ending (e.g., .com, .org). It points the resolver in the right direction.

TLD Name Server Narrows It Down: The TLD name server is like a regional map. It knows which authoritative name server is responsible for the specific domain you're looking for (e.g., example.com) and sends the resolver there.

Authoritative Name Server Delivers the Address: The authoritative name server is the final stop. It's like the street address of the website you want. It holds the correct IP address and sends it back to the resolver.

The DNS Resolver Returns the Information: The resolver receives the IP address and gives it to your computer. It also remembers it for a while (caches it), in case you want to revisit the website soon.

Your Computer Connects: Now that your computer knows the IP address, it can connect directly to the web server hosting the website, and you can start browsing.
```

Host file allows user to overide DNS cache and manually map to a domain. DNS concepts
```bash
DNS Concept	Description	Example
Domain Name	A human-readable label for a website or other internet resource.	www.example.com
IP Address	A unique numerical identifier assigned to each device connected to the internet.	192.0.2.1
DNS Resolver	A server that translates domain names into IP addresses.	Your ISP's DNS server or public resolvers like Google DNS (8.8.8.8)
Root Name Server	The top-level servers in the DNS hierarchy.	There are 13 root servers worldwide, named A-M: a.root-servers.net
TLD Name Server	Servers responsible for specific top-level domains (e.g., .com, .org).	Verisign for .com, PIR for .org
Authoritative Name Server	The server that holds the actual IP address for a domain.	Often managed by hosting providers or domain registrars.
DNS Record Types	Different types of information stored in DNS.	A, AAAA, CNAME, MX, NS, TXT, etc.
```
Record Types
```bash
A	Address Record	Maps a hostname to its IPv4 address.	www.example.com. IN A 192.0.2.1
AAAA	IPv6 Address Record	Maps a hostname to its IPv6 address.	www.example.com. IN AAAA 2001:db8:85a3::8a2e:370:7334
CNAME	Canonical Name Record	Creates an alias for a hostname, pointing it to another hostname.	blog.example.com. IN CNAME webserver.example.net.
MX	Mail Exchange Record	Specifies the mail server(s) responsible for handling email for the domain.	example.com. IN MX 10 mail.example.com.
NS	Name Server Record	Delegates a DNS zone to a specific authoritative name server.	example.com. IN NS ns1.example.com.
TXT	Text Record	Stores arbitrary text information, often used for domain verification or security policies.	example.com. IN TXT "v=spf1 mx -all" (SPF record)
SOA	Start of Authority Record	Specifies administrative information about a DNS zone, including the primary name server, responsible person's email, and other parameters.	example.com. IN SOA ns1.example.com. admin.example.com. 2024060301 10800 3600 604800 86400
SRV	Service Record	Defines the hostname and port number for specific services.	_sip._udp.example.com. IN SRV 10 5 5060 sipserver.example.com.
PTR	Pointer Record	Used for reverse DNS lookups, mapping an IP address to a hostname.	1.2.0.192.in-addr.arpa. IN PTR www.example.com.
```
Helpfull DNS enumeration tools
```bash
dig	Versatile DNS lookup tool that supports various query types (A, MX, NS, TXT, etc.) and detailed output.	Manual DNS queries, zone transfers (if allowed), troubleshooting DNS issues, and in-depth analysis of DNS records.
nslookup	Simpler DNS lookup tool, primarily for A, AAAA, and MX records.	Basic DNS queries, quick checks of domain resolution and mail server records.
host	Streamlined DNS lookup tool with concise output.	Quick checks of A, AAAA, and MX records.
dnsenum	Automated DNS enumeration tool, dictionary attacks, brute-forcing, zone transfers (if allowed).	Discovering subdomains and gathering DNS information efficiently.
fierce	DNS reconnaissance and subdomain enumeration tool with recursive search and wildcard detection.	User-friendly interface for DNS reconnaissance, identifying subdomains and potential targets.
dnsrecon	Combines multiple DNS reconnaissance techniques and supports various output formats.	Comprehensive DNS enumeration, identifying subdomains, and gathering DNS records for further analysis.
theHarvester	OSINT tool that gathers information from various sources, including DNS records (email addresses).	Collecting email addresses, employee information, and other data associated with a domain from multiple sources.
Online DNS Lookup Services	User-friendly interfaces for performing DNS lookups.	Quick and easy DNS lookups, convenient when command-line tools are not available, checking for domain availability or basic information
```
Dig Command Usage
```bash
Command	Description
dig domain.com	Performs a default A record lookup for the domain.
dig domain.com A	Retrieves the IPv4 address (A record) associated with the domain.
dig domain.com AAAA	Retrieves the IPv6 address (AAAA record) associated with the domain.
dig domain.com MX	Finds the mail servers (MX records) responsible for the domain.
dig domain.com NS	Identifies the authoritative name servers for the domain.
dig domain.com TXT	Retrieves any TXT records associated with the domain.
dig domain.com CNAME	Retrieves the canonical name (CNAME) record for the domain.
dig domain.com SOA	Retrieves the start of authority (SOA) record for the domain.
dig @1.1.1.1 domain.com	Specifies a specific name server to query; in this case 1.1.1.1
dig +trace domain.com	Shows the full path of DNS resolution.
dig -x 192.168.1.1	Performs a reverse lookup on the IP address 192.168.1.1 to find the associated host name. You may need to specify a name server.
dig +short domain.com	Provides a short, concise answer to the query.
dig +noall +answer domain.com	Displays only the answer section of the query output.
dig domain.com ANY	Retrieves all available DNS records for the domain (Note: Many DNS servers ignore ANY queries to reduce load and prevent abuse, as per RFC 8482).
```
# DNS Bruteforcing
DNS enum is really fast for DNS recon, it also allows recursion and does zone transfers by default. Good command to user
```bash
└─$ dnsenum --enum inlanefreight.com -f  /home/kali/SecLists-master/Discovery/DNS/subdomains-top1million-20000.txt    
```
Command Meanings
```
dnsenum --enum inlanefreight.com: We specify the target domain we want to enumerate, along with a shortcut for some tuning options --enum.
-f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt: We indicate the path to the SecLists wordlist we'll use for brute-forcing. Adjust the path if your SecLists installation is different.
-r: This option enables recursive subdomain brute-forcing, meaning that if dnsenum finds a subdomain, it will then try to enumerate subdomains of that subdomain.
```
# DNS Zones
DNS zones hold all of the alias's and hostnames of DNS records for a specific zone, and is a wealth of information. Typically a zone transfer is not allowed by a unknown IP and only can be triggered via valid DNS servers. That being said misconfigurations happen and the follwing DIG command can enumerate the DNS zone via conducting a transfer.
```bash
└─$ dig axfr inlanefreight.htb @10.129.144.236              
```
dig axfr (zone transfer) target @target_ip

# VOSTs
VHOST will allow a server to host multiple web pages on a single IP it will sever the correct content based on the web request header. Enumerating differnt domains in a VHOST can be done with Fuff
```bash
└─$ ffuf -w /home/kali/SecLists-master/Discovery/DNS/namelist.txt \
-u http://inlanefreight.htb:53295 -H "Host: FUZZ.inlanefreight.htb" \
-fs 116
```

# Website Service enumeration 
WaWappalyzer is realy great browser extension. Also nikto
```bash
└─$ nikto -h app.inlanefreight.local -Tuning b
```

# Web Crawling tool
A great tool for web crawling is ReconSpider
Intall -
```bash
darkmercinary7@htb[/htb]$ pip3 install scrapy
darkmercinary7@htb[/htb]$ wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
darkmercinary7@htb[/htb]$ unzip ReconSpider.zip 
```
Command to execute reconspider--
```bash
darkmercinary7@htb[/htb]$ python3 ReconSpider.py http://inlanefreight.com
```
Results from the scrape are saved in results.json and can have some good enumeration material in it.

# Google Dorking Cheatsheet
```
Operator	Operator Description	Example	Example Description
site:	Limits results to a specific website or domain.	site:example.com	Find all publicly accessible pages on example.com.
inurl:	Finds pages with a specific term in the URL.	inurl:login	Search for login pages on any website.
filetype:	Searches for files of a particular type.	filetype:pdf	Find downloadable PDF documents.
intitle:	Finds pages with a specific term in the title.	intitle:"confidential report"	Look for documents titled "confidential report" or similar variations.
intext: or inbody:	Searches for a term within the body text of pages.	intext:"password reset"	Identify webpages containing the term “password reset”.
cache:	Displays the cached version of a webpage (if available).	cache:example.com	View the cached version of example.com to see its previous content.
link:	Finds pages that link to a specific webpage.	link:example.com	Identify websites linking to example.com.
related:	Finds websites related to a specific webpage.	related:example.com	Discover websites similar to example.com.
info:	Provides a summary of information about a webpage.	info:example.com	Get basic details about example.com, such as its title and description.
define:	Provides definitions of a word or phrase.	define:phishing	Get a definition of "phishing" from various sources.
numrange:	Searches for numbers within a specific range.	site:example.com numrange:1000-2000	Find pages on example.com containing numbers between 1000 and 2000.
allintext:	Finds pages containing all specified words in the body text.	allintext:admin password reset	Search for pages containing both "admin" and "password reset" in the body text.
allinurl:	Finds pages containing all specified words in the URL.	allinurl:admin panel	Look for pages with "admin" and "panel" in the URL.
allintitle:	Finds pages containing all specified words in the title.	allintitle:confidential report 2023	Search for pages with "confidential," "report," and "2023" in the title.
AND	Narrows results by requiring all terms to be present.	site:example.com AND (inurl:admin OR inurl:login)	Find admin or login pages specifically on example.com.
OR	Broadens results by including pages with any of the terms.	"linux" OR "ubuntu" OR "debian"	Search for webpages mentioning Linux, Ubuntu, or Debian.
NOT	Excludes results containing the specified term.	site:bank.com NOT inurl:login	Find pages on bank.com excluding login pages.
* (wildcard)	Represents any character or word.	site:socialnetwork.com filetype:pdf user* manual	Search for user manuals (user guide, user handbook) in PDF format on socialnetwork.com.
.. (range search)	Finds results within a specified numerical range.	site:ecommerce.com "price" 100..500	Look for products priced between 100 and 500 on an e-commerce website.
" " (quotation marks)	Searches for exact phrases.	"information security policy"	Find documents mentioning the exact phrase "information security policy".
- (minus sign)	Excludes terms from the search results.	site:news.com -inurl:sports	Search for news articles on news.com excluding sports-related content.
```

# Automate recon Final Recon
Install
```bash
darkmercinary7@htb[/htb]$ git clone https://github.com/thewhiteh4t/FinalRecon.git
darkmercinary7@htb[/htb]$ cd FinalRecon
darkmercinary7@htb[/htb]$ sudo python3 -m pip install --break-system-packages -r requirements.txt
darkmercinary7@htb[/htb]$ chmod +x ./finalrecon.py
darkmercinary7@htb[/htb]$ ./finalrecon.py --help
```
Usage
```bash
darkmercinary7@htb[/htb]$ ./finalrecon.py --headers --whois --url http://inlanefreight.com
```
OR 

```bash
cd ~/Desktop/FinalRecon

# 1) make sure venv support is installed
sudo apt update
sudo apt install -y python3-venv

# 2) create + activate venv
python3 -m venv .venv
source .venv/bin/activate

# 3) upgrade pip + install/upgrade deps (especially tldextract)
python -m pip install -U pip setuptools wheel
python -m pip install -U "tldextract>=5.3.0"

# 4) install finalrecon requirements inside venv (no sudo)
pip install -r requirements.txt

# 5) run using the venv python
python finalrecon.py --full --url http://inlanefreight.com
```