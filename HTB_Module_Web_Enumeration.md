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