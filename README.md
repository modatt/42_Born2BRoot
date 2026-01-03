<div align="center">

# 🛡️ Born2beRoot

[![42 School](https://img.shields.io/badge/42-School-000000?style=for-the-badge&logo=42&logoColor=white)](https://42.fr)
[![Debian](https://img.shields.io/badge/Debian-A81D33?style=for-the-badge&logo=debian&logoColor=white)](https://www.debian.org/)
[![VirtualBox](https://img.shields.io/badge/VirtualBox-183A61?style=for-the-badge&logo=virtualbox&logoColor=white)](https://www.virtualbox.org/)

**A comprehensive system administration project focused on virtual machine configuration, security hardening, and server management.**

[Overview](#-overview) •
[Features](#-features) •
[Installation](#-installation) •
[Configuration](#%EF%B8%8F-configuration) •
[Usage](#-usage) •
[Documentation](#-documentation)

</div>

---

## 📋 Overview

**Born2beRoot** is a foundational system administration project from the 42 curriculum that challenges students to set up and configure a secure virtual machine from scratch. This project provides hands-on experience with Linux server administration, security policies, and automation.

### 🎯 Project Objectives

- Configure a virtual machine with strict security requirements
- Implement robust user management and access control
- Set up SSH, firewall, and encrypted partitions
- Create automated system monitoring solutions
- Develop understanding of system administration best practices

### 💻 Technical Stack

- **Operating System:** Debian (stable)
- **Virtualization:** VirtualBox
- **Security:** AppArmor, UFW, encrypted LVM
- **Access:** SSH (custom port configuration)
- **Monitoring:** Custom Bash script with cron automation

---

## ✨ Features

### 🔐 Security Implementation

#### SSH Configuration
- **Custom Port:** Configured to run on port 4242
- **Root Access:** Disabled for enhanced security
- **Key-based Authentication:** Supported for secure remote access

#### Firewall (UFW)
- **Default Policy:** Deny all incoming, allow all outgoing
- **Custom Rules:** Port 4242 allowed for SSH
- **Boot Persistence:** Enabled on system startup

#### Encrypted Storage
- **LVM Encryption:** Full disk encryption using LUKS
- **Partition Management:** Logical Volume Management for flexibility
- **Data Protection:** At-rest encryption for all data

### 👥 User Management

#### Password Policy
- **Expiration:** 30-day password lifetime
- **Minimum Length:** 10 characters
- **Complexity Requirements:**
  - At least one uppercase letter
  - At least one lowercase letter
  - At least one digit
  - No username inclusion
  - No more than 3 consecutive identical characters
- **History:** 5 previous passwords remembered

#### Access Control
- **Failed Login Protection:** Account lockout after 3 failed attempts
- **Group Management:** Custom groups (`sudo`, `user42`)
- **Privilege Escalation:** Strict sudo configuration

### ⚙️ Sudo Configuration

```bash
# Sudo security hardening
- Maximum password attempts: 3
- Custom error message for failed attempts
- Input/Output logging: /var/log/sudo/
- TTY requirement: Enabled
- Secure PATH restriction
- Archive logging of all sudo commands
```

### 📊 System Monitoring

Automated monitoring script (`monitoring.sh`) that displays:

- **Architecture:** OS and kernel version
- **Physical Processors:** CPU count
- **Virtual Processors:** vCPU count
- **Memory Usage:** RAM utilization and percentage
- **Disk Usage:** Storage consumption and percentage
- **CPU Load:** Current processor usage
- **Last Boot:** System uptime information
- **LVM Status:** Logical volume detection
- **TCP Connections:** Active connection count
- **User Sessions:** Logged-in user count
- **Network:** IP address and MAC address
- **Sudo Commands:** Total executed sudo commands

**Broadcast Interval:** Every 10 minutes via `wall` command

---

## 🚀 Installation

### Prerequisites

- VirtualBox (version 6.0 or higher)
- Debian ISO image (stable release)
- Minimum 8GB disk space
- 1GB RAM minimum (2GB recommended)

### Step-by-Step Setup

1. **Create Virtual Machine**
   ```bash
   # VirtualBox VM Settings
   - Name: Born2beRoot
   - Type: Linux
   - Version: Debian (64-bit)
   - Memory: 1024 MB
   - Hard Disk: 8 GB (VDI, Dynamically allocated)
   ```

2. **Install Debian**
   - Boot from Debian ISO
   - Select "Install" (not graphical install)
   - Follow installation prompts
   - Configure encrypted LVM partitions

3. **Partition Scheme**
   ```
   /boot      - 500 MB (ext4)
   LVM Group (encrypted):
     /        - 2.5 GB (ext4)
     swap     - 1 GB
     /home    - 1 GB (ext4)
     /var     - 1 GB (ext4)
     /srv     - 1 GB (ext4)
     /tmp     - 1 GB (ext4)
     /var/log - remaining space (ext4)
   ```

---

## ⚙️ Configuration

### 1. System Updates

```bash
su -
apt update && apt upgrade -y
```

### 2. Install Required Packages

```bash
apt install -y sudo ufw vim net-tools
```

### 3. User Setup

```bash
# Add user to sudo group
usermod -aG sudo <username>

# Create user42 group
groupadd user42
usermod -aG user42 <username>

# Verify group membership
groups <username>
```

### 4. SSH Configuration

```bash
# Edit SSH config
vim /etc/ssh/sshd_config

# Modify these settings:
Port 4242
PermitRootLogin no

# Restart SSH service
systemctl restart sshd
```

### 5. UFW Firewall

```bash
# Install and configure UFW
ufw allow 4242
ufw enable
ufw status verbose
```

### 6. Password Policy

```bash
# Edit login.defs
vim /etc/login.defs

# Set these values:
PASS_MAX_DAYS 30
PASS_MIN_DAYS 2
PASS_WARN_AGE 7

# Install libpam-pwquality
apt install libpam-pwquality

# Edit PAM configuration
vim /etc/pam.d/common-password

# Add/modify:
password requisite pam_pwquality.so retry=3 minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```

### 7. Sudo Configuration

```bash
# Edit sudoers file
visudo

# Add these lines:
Defaults  passwd_tries=3
Defaults  badpass_message="Custom error message"
Defaults  logfile="/var/log/sudo/sudo.log"
Defaults  log_input,log_output
Defaults  iolog_dir="/var/log/sudo"
Defaults  requiretty
Defaults  secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```

### 8. Monitoring Script

Create the monitoring script at `/usr/local/bin/monitoring.sh`:

```bash
#!/bin/bash

# System architecture
arch=$(uname -a)

# Physical CPUs
pcpu=$(grep "physical id" /proc/cpuinfo | sort -u | wc -l)

# Virtual CPUs
vcpu=$(grep "processor" /proc/cpuinfo | wc -l)

# RAM usage
ram_total=$(free -m | awk '$1 == "Mem:" {print $2}')
ram_used=$(free -m | awk '$1 == "Mem:" {print $3}')
ram_percent=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')

# Disk usage
disk_total=$(df -BG | grep '^/dev/' | grep -v '/boot$' | awk '{ft += $2} END {print ft}')
disk_used=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} END {print ut}')
disk_percent=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} {ft+= $2} END {printf("%d"), ut/ft*100}')

# CPU load
cpu_load=$(top -bn1 | grep '^%Cpu' | cut -c 9- | xargs | awk '{printf("%.1f%%"), $1 + $3}')

# Last boot
last_boot=$(who -b | awk '$1 == "system" {print $3 " " $4}')

# LVM check
lvm_use=$(if [ $(lsblk | grep "lvm" | wc -l) -eq 0 ]; then echo no; else echo yes; fi)

# TCP connections
tcp_conn=$(ss -Ht state established | wc -l)

# User log
user_log=$(users | wc -w)

# Network
ip=$(hostname -I)
mac=$(ip link show | grep "ether" | awk '{print $2}')

# Sudo commands
sudo_cmd=$(journalctl _COMM=sudo | grep COMMAND | wc -l)

wall "	#Architecture: $arch
	#CPU physical: $pcpu
	#vCPU: $vcpu
	#Memory Usage: $ram_used/${ram_total}MB ($ram_percent%)
	#Disk Usage: $disk_used/${disk_total}Gb ($disk_percent%)
	#CPU load: $cpu_load
	#Last boot: $last_boot
	#LVM use: $lvm_use
	#Connections TCP: $tcp_conn ESTABLISHED
	#User log: $user_log
	#Network: IP $ip ($mac)
	#Sudo: $sudo_cmd cmd"
```

Make the script executable and add to crontab:

```bash
chmod +x /usr/local/bin/monitoring.sh

# Edit root crontab
crontab -e

# Add this line:
*/10 * * * * /usr/local/bin/monitoring.sh
```

---

## 📖 Usage

### Verify Configuration

```bash
# Check SSH status
systemctl status ssh

# Check firewall rules
ufw status verbose

# Check user groups
groups <username>

# Verify sudo logging
cat /var/log/sudo/sudo.log

# Test monitoring script
bash /usr/local/bin/monitoring.sh
```

### Managing the VM

```bash
# Check LVM configuration
lvdisplay
pvdisplay
vgdisplay

# Monitor system resources
htop

# View system logs
journalctl -xe
```

### Common Operations

```bash
# Add new user
sudo adduser <username>
sudo usermod -aG sudo <username>
sudo usermod -aG user42 <username>

# Change password
sudo passwd <username>

# Check password policy for user
sudo chage -l <username>

# Modify firewall rules
sudo ufw allow <port>
sudo ufw deny <port>
sudo ufw delete allow <port>

# Stop/disable monitoring script
sudo crontab -e  # Comment out the monitoring line
```

---

## 📚 Documentation

### Key Concepts

#### Why Debian?
- **Stability:** Long-term support and reliability
- **Security:** Regular security updates and patches
- **APT Package Manager:** Robust dependency management
- **Community:** Extensive documentation and support
- **Lightweight:** Minimal resource requirements
- **Industry Standard:** Widely used in server environments

#### AppArmor vs SELinux
| Feature | AppArmor | SELinux |
|---------|----------|---------|
| **Type** | Path-based MAC | Label-based MAC |
| **Complexity** | Easier to configure | More complex |
| **Default on Debian** | ✅ Yes | ❌ No |
| **Learning Curve** | Gentler | Steeper |
| **Use Case** | Good for beginners | Enterprise environments |

#### APT vs Aptitude
| Feature | APT | Aptitude |
|---------|-----|----------|
| **Interface** | Command-line only | CLI + Text UI |
| **Speed** | Faster | Slightly slower |
| **Dependencies** | Basic resolution | Advanced resolution |
| **Best For** | Scripting | Interactive use |
| **Removal** | Leaves dependencies | Auto-removes unused |

### Useful Commands

```bash
# Security & System
aa-status                    # Check AppArmor status
sudo aa-enforce /path/to/profile  # Enforce AppArmor profile
lsblk                        # View partition information
df -h                        # Disk space usage
free -h                      # Memory usage

# User Management
chage -l <username>          # Check password expiration
passwd -S <username>         # Check password status
getent passwd <username>     # View user information
id <username>                # Check user ID and groups

# Network & SSH
ss -tunlp                    # Monitor network connections
netstat -tulpn               # Alternative network monitor
systemctl status sshd        # SSH service status
journalctl -u ssh            # SSH service logs

# Firewall
ufw status numbered          # List rules with numbers
ufw delete <number>          # Delete rule by number
ufw reload                   # Reload firewall

# Sudo & Logs
tail -f /var/log/sudo/sudo.log    # Monitor sudo logs
journalctl -f                     # Follow system logs
lastlog                           # Last login information
last                              # Login history
```

### Troubleshooting

#### SSH Connection Issues
```bash
# Check if SSH is running
systemctl status ssh

# Verify SSH port
grep "^Port" /etc/ssh/sshd_config

# Check firewall
ufw status | grep 4242

# Test connection locally
ssh localhost -p 4242
```

#### Sudo Permission Denied
```bash
# Verify user in sudo group
groups <username>

# Re-add to sudo group if needed
usermod -aG sudo <username>

# User needs to logout/login for group changes
```

#### Monitoring Script Not Running
```bash
# Check crontab
crontab -l

# Verify script permissions
ls -l /usr/local/bin/monitoring.sh

# Test script manually
bash /usr/local/bin/monitoring.sh

# Check cron logs
grep CRON /var/log/syslog
```

---

## 🏆 Project Requirements Checklist

- [x] Virtual machine running Debian (latest stable)
- [x] Encrypted partitions using LVM
- [x] SSH running on port 4242
- [x] UFW firewall configured and enabled
- [x] Root login disabled via SSH
- [x] Strong password policy implemented
- [x] Sudo configuration with logging
- [x] User management with custom groups
- [x] Monitoring script with cron automation
- [x] AppArmor security enabled
- [x] No graphical environment
- [x] Hostname configured properly

---

## 🔍 Evaluation Questions & Answers

<details>
<summary><b>How does a virtual machine work?</b></summary>

A virtual machine (VM) is a software emulation of a physical computer. It runs on a host operating system and provides a complete computing environment with virtualized hardware resources (CPU, RAM, disk, network). The hypervisor (like VirtualBox) manages these resources and isolates the VM from the host.
</details>

<details>
<summary><b>Why did you choose Debian?</b></summary>

Debian was chosen for its stability, security, ease of use, and strong community support. It's lightweight, has excellent documentation, and uses APT for package management. It's also widely used in server environments, making it ideal for learning system administration.
</details>

<details>
<summary><b>Difference between Debian and Rocky/CentOS?</b></summary>

- **Debian:** Uses APT, .deb packages, community-driven, more frequent releases
- **Rocky/CentOS:** Uses DNF/YUM, .rpm packages, enterprise-focused, longer release cycles
- Both are stable, but Debian is generally easier for beginners
</details>

<details>
<summary><b>What is the purpose of LVM?</b></summary>

LVM (Logical Volume Manager) provides flexible disk management:
- Easy resizing of partitions without downtime
- Snapshot capabilities for backups
- Better disk space utilization
- Ability to span volumes across multiple disks
</details>

<details>
<summary><b>What is AppArmor?</b></summary>

AppArmor is a Linux security module that implements Mandatory Access Control (MAC). It restricts programs' capabilities based on profiles, limiting what files they can access and what operations they can perform, adding an extra layer of security beyond traditional user permissions.
</details>

<details>
<summary><b>What is UFW and how does it work?</b></summary>

UFW (Uncomplicated Firewall) is a user-friendly interface for managing iptables firewall rules. It simplifies firewall configuration with intuitive commands, allowing easy control over which ports and services can accept connections.
</details>

<details>
<summary><b>What is SSH and why use port 4242?</b></summary>

SSH (Secure Shell) is a cryptographic network protocol for secure remote access. Using port 4242 instead of the default port 22 provides security through obscurity, reducing automated attacks from bots scanning for default SSH ports.
</details>

---

## 📝 Additional Resources

- [Debian Official Documentation](https://www.debian.org/doc/)
- [Linux System Administration Guide](https://linux.die.net/)
- [SSH Security Best Practices](https://www.ssh.com/academy/ssh/security)
- [UFW Guide](https://help.ubuntu.com/community/UFW)
- [LVM Administration Guide](https://tldp.org/HOWTO/LVM-HOWTO/)

---

## 🤝 Contributing

This project is part of the 42 School curriculum and is primarily for educational purposes. However, suggestions for improvements are welcome!

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Open a Pull Request

---

## 📝 License

This project is created for educational purposes as part of the 42 curriculum. Feel free to use it as a reference for your own learning.

---

## 👤 Author

Created with modat by a 42 student

**Project Completion Date:** January 2026

---

<div align="center">

### ⭐ If this helped you, please consider giving it a star!

**[⬆ Back to Top](#-born2beroot)**

---

*"The only way to learn a new programming language is by writing programs in it." - Dennis Ritchie*

</div>
