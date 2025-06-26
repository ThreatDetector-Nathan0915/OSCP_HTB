# 🧠 Linux Prompt Symbols + Bash Customization + Command Cheat Sheet

---

## 🔑 Shell Prompt Symbols

| Symbol | Meaning        |
|--------|----------------|
| `#`    | You are **root** (administrator) |
| `$`    | You are a **normal user**        |

These symbols appear at the start of your shell prompt and help you instantly know your privilege level.

---

## 🎨 Customize Your Shell Prompt with `.bashrc`

The `.bashrc` file lets you change how your terminal looks and behaves.

### 🔧 To edit:
```bash
nano ~/.bashrc
```

Add or modify the `PS1` variable to change your prompt style.

### 💡 Online Bash Prompt Generator:
Create your own colored/custom bash prompt easily at:  
🔗 [https://bash-prompt-generator.org](https://bash-prompt-generator.org)

---

## 📚 Linux Command Cheat Sheet

Here's a list of commonly used Linux commands grouped by category:

### 📂 File and Directory Management

```bash
ls              # List directory contents
pwd             # Print working directory
cd <dir>        # Change directory
mkdir <dir>     # Make new directory
rm <file>       # Remove file
rm -r <dir>     # Remove directory and contents
cp <src> <dest> # Copy file or directory
mv <src> <dest> # Move or rename file
touch <file>    # Create empty file
cat <file>      # View file contents
```

### 🔍 Searching and Finding

```bash
find / -name <file>        # Find files by name
grep "text" <file>         # Search text in file
grep -r "text" <directory> # Recursive grep
locate <filename>          # Find file (uses database)
```

### 🧠 Permissions and Ownership

```bash
chmod +x <file>        # Make file executable
chmod 755 <file>       # Set permissions
chown user:user <file> # Change ownership
```

### 🖥️ System Info

```bash
uname -a         # All system info
hostname         # Show hostname
df -h            # Disk space usage
free -h          # RAM usage
uptime           # System uptime
top              # Active processes
whoami           # Current user
id               # Current user ID info
```

### 🔧 Process and Service Control

```bash
ps aux                # Show all running processes
kill <pid>            # Kill process by PID
killall <name>        # Kill process by name
systemctl status      # Show system service status
systemctl restart ssh # Restart a service (e.g., SSH)
```

### 📦 Package Management (Debian/Ubuntu)

```bash
sudo apt update            # Refresh package lists
sudo apt install <package> # Install a package
sudo apt remove <package>  # Remove a package
dpkg -l                    # List installed packages
```

### 🔐 User Management

```bash
adduser <username>     # Add new user
passwd <username>      # Set user password
usermod -aG sudo <user> # Add user to sudo group
```

### 🌐 Networking

```bash
ip a                 # Show IP addresses
ping <host>          # Ping a host
netstat -tulnp       # Show listening ports
ss -tuln             # Faster netstat alternative
curl <url>           # HTTP requests
wget <url>           # Download files
```

---

## ✅ Summary

| Concept       | Description                                    |
|---------------|------------------------------------------------|
| `#` prompt    | Root shell                                     |
| `$` prompt    | Normal user shell                              |
| `.bashrc`     | Config file for shell behavior/customization   |
| `PS1`         | Variable that sets how your prompt looks       |
| Generator     | [bash-prompt-generator.org](https://bash-prompt-generator.org) |
| Cheat Sheet   | Common Linux commands for daily operations     |

---

Use this sheet as a quick reference while navigating or customizing your Linux system.
