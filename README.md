# Born2beRoot - Complete System Administration Guide

## Table of Contents
- [Why I Chose Debian](#why-i-chose-debian)
- [Virtualization](#virtualization)
- [Hypervisors](#hypervisors)
  - [Type 1: Bare Metal](#type-1-bare-metal)
  - [Type 2: Hosted](#type-2-hosted)
- [LVM and Disk Partitioning](#lvm-and-disk-partitioning)
  - [What is LVM?](#what-is-lvm)
  - [Why Use LVM?](#why-use-lvm)
  - [Primary Partitions vs LVM](#primary-partitions-vs-lvm)
  - [The Encryption Layer](#the-encryption-layer)
  - [LVM Architecture](#lvm-architecture---the-three-levels)
  - [Your System's Logical Volumes](#your-systems-logical-volumes-explained)
  - [Visual Hierarchy](#visual-hierarchy)
  - [Key Concepts](#key-concepts)
- [Server](#server)
- [APT and Aptitude](#apt-and-aptitude)
  - [APT](#apt)
  - [Aptitude](#aptitude)
- [SSH](#ssh)
  - [Change Default Port](#change-the-default-port-of-ssh)
  - [Hostname Configuration](#hostname-configuration)
- [AppArmor Security](#apparmor-security)
- [Firewalls](#firewalls)
  - [Linux UFW](#linux-ufw)
- [Password Policies](#password-policies)
  - [Password Expiration Settings](#1-password-expiration-settings)
  - [Password Quality Requirements](#2-password-quality-requirements)
  - [Apply to Existing Users](#3-apply-expiration-settings-to-existing-users)
- [Superuser Do Configuration](#superuser-do-configuration)
  - [Add/Remove Users from Sudoers](#add-or-remove-a-user-from-sudoers)
  - [Using Visudo](#how-to-use-visudo)
  - [Born2beRoot Requirements](#born2beroot-requirements)
  - [Configure New User](#configure-new-user)
- [Monitoring Script](#monitoring-script)
  - [Essential System Utilities](#essential-system-utilities-for-the-script)
  - [What is Cron](#what-cron-is)
- [WordPress Setup Guide](#super-simple-wordpress-setup-guide)
  - [Install Web Server](#step-1-install-the-web-server)
  - [Configure Firewall](#step-2-open-the-door-for-visitors)
  - [Install Database](#step-3-install-the-database)
  - [Secure Database](#step-4-make-the-database-secure)
  - [Create WordPress Database](#step-5-create-a-space-for-wordpress)
  - [Install PHP](#step-6-install-php-wordpresss-language)
  - [Download WordPress](#step-7-download-wordpress)
  - [Connect to Database](#step-8-connect-wordpress-to-the-database)
  - [Enable FastCGI](#step-9-make-everything-work-together)
  - [Finish Setup](#step-10-finish-the-setup)
  - [Troubleshooting](#quick-troubleshooting)
  - [Bonus Service](#bonus-service---monitorix)

---

## Why I Chose Debian

> **Personal Choice**

It's a friendly-use operating system for setting up a server easily. From the instructions, they tell that Rocky needs more technical experience. This is simply why I prefer Debian, and it's been a long time since I started using Debian in my Kali Linux laptop.

### CentOS vs Debian

| Feature             | CentOS                                    | Debian                                 |
| ------------------- | ----------------------------------------- | -------------------------------------- |
| **Linux Family**    | RHEL (Red Hat Enterprise Linux)           | Independent, community-driven          |
| **Based on**        | RHEL source code                          | Original distribution                  |
| **Target Audience** | Enterprise servers, stability-focused     | Servers, desktops, and community users |
| **Use**             | Enterprise, web hosting, professional use | Community projects, servers, education |

---

## Virtualization

> **What is Virtualization?**

**Virtualization** is a technology that lets you run **multiple operating systems** on a _single_ physical computer **at the same time**.

A **Virtual Machine (VM)** is a _fake computer_ that runs inside your real computer.

**In simple terms:**
- **Virtualization** = technique to create isolated computers inside your real computer
- **Virtual Machine** = one of those isolated computers

---

## Hypervisors

> **Definition**

A **hypervisor** is a piece of software that enables a user to create and run one or more virtual machines simultaneously. A hypervisor is also known as the **virtual machine monitor (VMM)** and controls the resources of the host machine and allocates to each VM the resources it needs (memory, CPU...), making sure that these VMs do not interfere with each other.

### Type 1: Bare Metal

**Installed directly on the hardware**, it means:
- The hypervisor is installed **on the disk**, just like an operating system
- It boots **without any host OS**, and becomes the **first layer that controls the hardware**

**Boot process looks like:**
```
BIOS/UEFI → loads hypervisor → hypervisor controls hardware → runs VMs
```

### Type 2: Hosted

**Installed on an existing OS** like any normal program: VMware, VirtualBox

- On Linux: via `.deb`, `.rpm`, AppImage, etc.
- On Windows: `.exe` installer
- On macOS: `.dmg` application

---

## LVM and Disk Partitioning

### What is LVM?

**LVM (Logical Volume Manager)** is a flexible disk management system for Linux. Think of it as a layer between your physical hard drives and your file system that makes managing storage much easier.

### Why Use LVM?

- **Resize volumes** without unmounting (grow or shrink partitions on the fly)
- **Move data** between physical drives without downtime
- **Create snapshots** for backups
- **Combine multiple disks** into one large storage pool

### Primary Partitions vs LVM

#### Primary Partitions (sda1, sda2)

These are traditional, fixed-size partitions created directly on the physical disk.

**In your system:**
- **`/boot` (sda1 - 500MB)**: Contains kernel and boot files
  - **Why separate?** Because the **BIOS** can't read from logical partitions, and GRUB bootloader needs a simple, unencrypted partition it can read before the OS loads
  - Must be outside LVM/encryption so the system can boot
- **`sda2` (1KB)**: Extended partition container (holds sda5)

### The Encryption Layer

#### sda5_crypt (LUKS Encrypted Container)

Before LVM comes into play, your main partition (sda5) is **encrypted** using LUKS (Linux Unified Key Setup).

```
Physical Disk → Encryption (sda5_crypt) → LVM → Logical Volumes
```

This means all your data is protected. Even if someone steals your hard drive, they can't read your data without the encryption password.

### LVM Architecture - The Three Levels

LVM has three main layers that work together:

#### 1. PV (Physical Volume) - The Foundation
- **What it is:** A physical disk or partition prepared for LVM use
- **In your system:** `sda5_crypt` (the encrypted partition)
- **Identifier:** `254:0`
- **Think of it as:** The raw storage space available

#### 2. VG (Volume Group) - The Storage Pool
- **What it is:** A collection of Physical Volumes combined into one storage pool
- **In your system:** `LVMGroup`
- **Think of it as:** A flexible storage container that can grow by adding more PVs

#### 3. LV (Logical Volume) - The Usable Partitions
- **What it is:** Virtual partitions carved out of the Volume Group
- **In your system:** root, swap, home, var, srv, tmp, var-log
- **Think of it as:** Like traditional partitions, but flexible and resizable

### Your System's Logical Volumes Explained

| Volume                | Size  | Mount Point | Purpose                                                    |
| --------------------- | ----- | ----------- | ---------------------------------------------------------- |
| **LVMGroup-root**     | 10GB  | `/`         | Core operating system files (binaries, libraries, configs) |
| **LVMGroup-swap**     | 2.3GB | `[SWAP]`    | Virtual memory for RAM overflow                            |
| **LVMGroup-home**     | 5GB   | `/home`     | User files, documents, personal data                       |
| **LVMGroup-var**      | 3GB   | `/var`      | Variable data (databases, caches, spool files)             |
| **LVMGroup-srv**      | 3GB   | `/srv`      | Service data (web servers, FTP, etc.)                      |
| **LVMGroup-tmp**      | 3GB   | `/tmp`      | Temporary files (cleared on reboot)                        |
| **LVMGroup-var--log** | 4GB   | `/var/log`  | System and application logs                                |

### Visual Hierarchy

```
sda (30.8GB Physical Disk)
│
├── sda1 (500MB) → /boot [Primary Partition]
│   └── Bootloader & Kernel files
│
├── sda2 (1KB Extended Partition Container)
│
└── sda5 (30.3GB) → Encrypted with LUKS
    └── sda5_crypt (Decrypted Device)
        └── PV (Physical Volume)
            └── VG: LVMGroup (Volume Group)
                ├── LV: root (10GB) → /
                ├── LV: swap (2.3GB) → [SWAP]
                ├── LV: home (5GB) → /home
                ├── LV: var (3GB) → /var
                ├── LV: srv (3GB) → /srv
                ├── LV: tmp (3GB) → /tmp
                └── LV: var--log (4GB) → /var/log
```

### Key Concepts

#### MAJ:MIN Numbers
- **Major:Minor device numbers** that the kernel uses to identify devices
- `8:0` = sda (main disk)
- `254:x` = dm (device-mapper) devices (LVM/encryption)

#### RM (Removable)
- `0` = Not removable (internal drive)
- `1` = Removable (USB, CD/DVD)

#### RO (Read-Only)
- `0` = Read/Write enabled
- `1` = Read-only mode

---

## Server

> **Definition**

A **server** is a computer — often with stronger resources — that **responds to requests from client devices** by providing specific services such as websites, email, DNS, files, databases, or applications.

---

## APT and Aptitude

### APT

**APT** = **Advanced Package Tool**

It is the **package manager** used by Debian-based Linux systems (Ubuntu, Kali, Linux Mint, Pop!_OS…).

**Main Roles:**
- Download software packages
- Install them
- Update them
- Remove them: `sudo apt remove [package]`
- Manage all their dependencies automatically

#### What is a Package?

A **package** is a `.deb` file that contains:
- The program
- Metadata
- Scripts (install, remove)
- Its dependencies

### Aptitude

A more advanced, menu-based version of apt. It can be used **from the command line** or with a **text-based GUI**.

**Some command line examples:**
```bash
sudo aptitude upgrade
sudo aptitude full-upgrade
sudo aptitude install nmap
```

#### Example of Aptitude Conflict Handling:

**APT:**
- Usually picks **one automatic solution**
- Example: "Remove package B to install A"

**Aptitude:**
- Gives **multiple choices**
- Example:
  1. Remove B
  2. Install older version of A
  3. Cancel the operation

---

## SSH

> **Secure Shell Protocol**

**SSH** is a secure protocol that lets you **remotely connect to another computer** through an encrypted terminal. You can use it to run commands, manage servers, transfer files, and administer machines safely over a network.

```bash
sudo apt install openssh-server
```

### Change the Default Port of SSH

```bash
sudo nano /etc/ssh/sshd_config
```

> **⚠️ Important Configuration**  
> Change Port 22 to Port 4242 and set PermitRootLogin to no. Remember to uncomment the lines after making changes.

```
Port 4242
.
.
.
PermitRootLogin no
```

**Start the service:**
```bash
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl restart ssh
```

**See IP address to connect from another machine:**
```bash
hostname -I
```

**Check SSH status:**
```bash
sudo systemctl status sshd
```

**Verify the listening port on the server:**
```bash
sudo ss -tulpn | grep ssh
```

### Hostname Configuration

> **What is Hostname?**

A **hostname** is the name of a device on a network.

**See your hostname:**
```bash
hostname
```

**See the IP address of your machine:**
```bash
hostname -I
```

**Change the hostname:**
```bash
sudo hostnamectl set-hostname newhostname
```

---

## AppArmor Security

> **Linux Security Module**

AppArmor is a **Linux security module** (LSM) that **limits application kernel access**. AppArmor is like a firewall, but for applications instead of network traffic.

### Just To Know

#### What a Profile Is
A file that defines permissions for an application (file access, network, capabilities).

> **📝 Note**  
> Starting from **Debian 10 (Buster)** and later (11, 12, now 13), **AppArmor is installed and enabled by default**.

#### Where Profiles Are Located
```bash
/etc/apparmor.d/
```

#### Check AppArmor Status
```bash
sudo aa-status
```

#### How to Check systemd Service
```bash
sudo systemctl status apparmor
```

---

## Firewalls

> **Network Security System**

A firewall is a security system that controls network traffic and decides which IP addresses and which ports should be accessed. In general, it protects your private network from unauthorized connections.

### It Takes These Actions:
- **ACCEPT** → let packet pass
- **DROP** → silently drop packet (no response)
- **REJECT** → block and send "connection refused"

### Linux UFW

**UFW (Uncomplicated Firewall)** is a simple command-line tool on Linux (mainly Ubuntu/Debian) used to configure the system firewall easily.

> **Important Note**  
> UFW is not a firewall itself, it's just a tool to control Linux system firewall. When you use UFW, you're really just telling Netfilter what rules to enforce, but in a much simpler way than writing iptables commands directly.

#### See Active Rules
```bash
sudo ufw status
```

#### Install if Not Installed
```bash
sudo apt install ufw
```

#### Enable/Disable
```bash
sudo ufw enable    # or disable
```

#### Allow Port
```bash
sudo ufw allow 22
```

#### Deny
```bash
sudo ufw deny 23
```

#### Delete a Rule
```bash
sudo ufw delete deny 10.13.100.13
sudo ufw delete deny from 10.13.100.13
```

#### Allow/Deny All In/Out
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

#### Enable Logging
Logging creates detailed records of network traffic (allowed and denied) that passes through a firewall, helping with network security, troubleshooting, and compliance.

```bash
sudo ufw logging on
```

**To see logs:**
```bash
journalctl -u ufw
journalctl | grep ufw
```

---

## Password Policies

### 1. Password Expiration Settings

Go to the **`/etc/login.defs`** configuration file and modify the following lines:

```bash
PASS_MAX_DAYS 30  # Make password expire after 30 days
PASS_MIN_DAYS 2   # Minimum days before password modification
PASS_WARN_AGE 7   # Number of days warning before password expires
```

> **⚠️ Important**  
> The changes will be applied to new users only. To ensure that the changes you made are applied to current users, use the `chage` command with `-M`, `-m`, `-W`.

```bash
sudo chage -M 30 username
sudo chage -m 2 username
sudo chage -W 7 username
```

### 2. Password Quality Requirements

To strengthen the password policy, we will utilize a module called **pwquality**:

```bash
sudo apt install libpam-pwquality
```

#### Configure

Go to the file:
```bash
sudo vim /etc/pam.d/common-password
```

Find this line:
```
password requisite pam_pwquality.so
```

Edit it to be:
```
password requisite pam_pwquality.so retry=3 minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 reject_username enforce_for_root

password requisite pam_pwquality.so retry=3 minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7
```

> **Note**  
> We added all rules except the `difok` to the root as well. The second line we added `difok` but not applied to root.

#### Password Rules Explained

| Rule              | Description                                               |
| ----------------- | --------------------------------------------------------- |
| `minlen=10`       | Minimum 10 characters                                     |
| `ucredit=-1`      | Requires 1 uppercase                                      |
| `lcredit=-1`      | Requires 1 lowercase                                      |
| `dcredit=-1`      | Requires 1 digit                                          |
| `ocredit=-1`      | Requires 1 special character                              |
| `retry=3`         | Allows 3 attempts                                         |
| `reject_username` | Avoid username in password                                |
| `difok=7`         | At least 7 characters different from the previous password|
| `maxrepeat=3`     | No more than 3 repetitions of a character in the password |

### 3. Apply Expiration Settings to Existing Users

For each user:
```bash
sudo chage -M 30 -m 2 -W 7 username
```

**Check:**
```bash
sudo chage -l username
```

**Expected output:**
```
Last password change                                    : Nov 28, 2025
Password expires                                        : Dec 28, 2025
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 2
Maximum number of days between password change          : 30
Number of days of warning before password expires       : 7
```

---

## Superuser Do Configuration

> **What is Sudo?**

It allows a normal user to run commands with root (administrator) privileges, without logging in as root. `sudo` is a program and it uses a special group (sudo) to control who can use it.

### Change Time for Test Issues

```bash
timedatectl set-time yyyy-mm-dd
```

### Add or Remove a User from Sudoers

**Add:**
```bash
usermod -aG sudo username
```

**Delete:**
```bash
sudo deluser user group
```

**Check membership:**
```bash
groups username
```

**Sudoers ASCII file:**
```bash
/etc/sudoers
```

**Direct access with root user:**
```bash
visudo
```

### How to Use Visudo

Run `visudo`, you will see something like this:

```
root    ALL=(ALL:ALL) ALL
%sudo   ALL=(ALL:ALL) ALL
```

This means:
- **root** can run everything
- **members of sudo group** can run everything with sudo

### Adding a Specific User with Privilege

Example: Give user `ahmed` full sudo rights:
```
ahmed ALL=(ALL:ALL) ALL
```

### Born2beRoot Requirements

#### The User Must NOT Be Root

You must create a **new user** (often called _your_login_) and give it sudo rights.

Example:
```bash
sudo usermod -aG sudo your_login
```

#### SUDO Must Ask for a Password Every Time

The subject _requires_ that every `sudo` command asks for password.

Inside `visudo`:
```
Defaults        passwd_tries=3
Defaults        badpass_message="Wrong password"
Defaults        logfile="/var/log/sudo/sudo.log"
Defaults        log_input,log_output
Defaults        iolog_dir="/var/log/sudo"
Defaults        requiretty
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
Defaults        timestamp_timeout=0
```

> **Important Configuration**  
> **`requiretty`**: This setting means sudo commands can ONLY be run from an actual TTY (interactive terminal). This prevents background scripts from using sudo privileges.

Most important is:
```
Defaults        timestamp_timeout=0
```

By default, `timestamp_timeout=15` means you can use sudo without re-entering the password for 15 minutes. `0` means each time you use sudo, you will enter the password.

**Sudo will not create the directory automatically:**
```bash
sudo mkdir -p /var/log/sudo
```

### Configure New User

After using:
```bash
sudo useradd -m newuser
```

**Give the new user a password:**
```bash
sudo passwd newuser
```

**Configure its shell if needed:**
```bash
sudo usermod -s /bin/bash username
```

---

## Monitoring Script

### Understanding System Commands

#### `uname -a` Command

```
Linux achahi42 6.1.0-41-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.158-1 (date) x86_64 GNU/Linux
```

- `Linux` - the name of the kernel
- `achahi` - the hostname
- `6.1.0-41-amd` - the kernel version
- `debian 6.1.158-1` - base system distribution

#### `lscpu` Command

This command displays more details about the CPU.

#### `ss` Command

It stands for **socket statistics**.
- `-a` option to show all sockets
- `-t` show TCP connections

### Essential System Utilities for the Script

#### `bc` (Basic Calculator)
Command-line calculator for mathematical operations and floating-point calculations in scripts.

#### `sysstat` (System Statistics)
Collection of performance monitoring tools (sar, iostat, mpstat) to track CPU, memory, and I/O usage.

#### Installation
```bash
sudo apt install bc sysstat
```

### Important Notes

> **Remember**  
> We excluded the `/boot` partition because it's not user-accessible storage.

> **Remember**  
> We use the second sample of the output because the first one is not for live statements.

### Understanding journalctl

**`journalctl`** is the tool/command set that lets you read, filter, and query logs stored in systemd's binary format.

`_COMM=name` makes it able to filter its output to just the command names.

### What Cron Is

> **Automated Task Scheduler**

- **`cron`** is a Linux utility to run scripts or commands **automatically at scheduled times**
- **You do not need "all terminals"** — cron runs tasks in the background, independent of terminal sessions

#### To Edit It:
```bash
crontab -e
```

It opens a file where you can add your script. Add this at the end of the file:

```bash
*/10 * * * * /path/to/your/script.sh
```

Each asterisk represents a timeline. Just add a slash and type a number after:

```
* * * * * command_to_run
│ │ │ │ │
│ │ │ │ └─ day of week (0-7, Sunday=0 or 7)
│ │ │ └── month (1-12)
│ │ └─── day of month (1-31)
│ └──── hour (0-23)
└───── minute (0-59)
```

---

## Super Simple WordPress Setup Guide

### STEP 1: Install the Web Server

```bash
sudo apt install lighttpd
```

> **What just happened?**  
> You installed a program that shows websites to people on the internet. **lighttpd** is a web server that manages client requests, listens for visitors, and shows them web pages.

### STEP 2: Open the Door for Visitors

```bash
sudo ufw allow 80
```

You told the firewall to let people visit your website.

### STEP 3: Install the Database

WordPress needs a place to save your posts and settings.

```bash
sudo apt install mariadb-server
```

You installed a database (like a filing system - storage space for your website's information).

### STEP 4: Make the Database Secure

```bash
sudo mysql_secure_installation
```

You'll be asked some questions. Here's what to answer:

- **Use unix authentication?** → Type `N` and press Enter
- **Change root password?** → Type `N` and press Enter
- **Remove anonymous users?** → Type `Y` and press Enter
- **Disallow root remote login?** → Type `Y` and press Enter
- **Remove test database?** → Type `Y` and press Enter
- **Reload privileges?** → Type `Y` and press Enter

### STEP 5: Create a Space for WordPress

Now we'll create a special storage area for WordPress.

**Copy each line one at a time:**

```bash
sudo mariadb
```

You're now inside the database. Copy these commands **one by one**:

```sql
CREATE DATABASE my_wordpress;
SHOW DATABASES;
```

```sql
GRANT ALL ON my_wordpress.* TO 'wpuser'@'localhost' IDENTIFIED BY 'ChangeThisPassword123!' WITH GRANT OPTION;
```

> **⚠️ IMPORTANT**  
> Change `ChangeThisPassword123!` to your own password!

```sql
FLUSH PRIVILEGES;
```

```sql
exit;
```

You created a storage box called "my_wordpress" and gave WordPress a key to access it.

### STEP 6: Install PHP (WordPress's Language)

```bash
sudo apt install php-cgi php-mysql
```

**php-cgi**: PHP Common Gateway Interface, is a PHP script interpreter because web services use PHP in backend like WordPress pages.

WordPress is written in PHP, so you installed the translator that makes it work.

### STEP 7: Download WordPress

```bash
sudo apt install wget
```

```bash
cd /var/www/
sudo wget http://wordpress.org/latest.tar.gz
```

```bash
sudo tar -xzvf latest.tar.gz
```

```bash
sudo rm latest.tar.gz
```

```bash
# Rename the html file to another name and give the name to the wordpress dir
sudo mv html/ html_back
sudo mv wordpress/ html
```

```bash
sudo chmod -R 755 html
```

> **Note**  
> `-R` = Recursive - It means the command applies **to the folder and everything inside it**.

You downloaded WordPress and put it in the right place.

### STEP 8: Connect WordPress to the Database

Always inside `/var/www/html`:

```bash
sudo cp wp-config-sample.php wp-config.php
```

```bash
sudo vim wp-config.php
```

A text editor will open. Find these three lines and change them:

**Change this** with the MariaDB database name, user, and password:

```php
define( 'DB_NAME', 'database_name_here' );
define( 'DB_USER', 'username_here' );
define( 'DB_PASSWORD', 'password_here' );
```

Save and exit.

### STEP 9: Make Everything Work Together

```bash
sudo lighty-enable-mod fastcgi-php
```

```bash
sudo service lighttpd force-reload
```

> **What just happened?**  
> You connected PHP to your web server.

### STEP 10: Finish the Setup

Open your web browser and go to:

```
http://YOUR_SERVER_IP
```

You should see WordPress asking you to:

1. Choose your language
2. Create your admin account
3. Name your website

**Follow the steps on screen and you're done!**

### Log In to Your Website

After setup, visit:

```
http://YOUR_SERVER_IP/wp-admin
```

Use the admin username and password you just created.

### Success Checklist

- [ ] Can you see the WordPress setup screen?
- [ ] Can you create your admin account?
- [ ] Can you log in to `/wp-admin`?

If you answered YES to all three → **Congratulations! Your WordPress site is live!** 🎊

### Quick Troubleshooting

**Can't connect to database?**  
Go back to Step 8 and double-check your password matches Step 5.

**Can't access the website?**  
Make sure you completed Step 2 (firewall) and your server IP is correct.

**See a blank page?**  
Run Step 9 again to restart the server.

### Bonus Service - Monitorix

> **What Next?**

I chose a service that displays the state of your service and resources used by the server: **Monitorix**.

**Installation:**
```bash
sudo apt install monitorix -y
```

**Enable service:**
```bash
sudo systemctl restart monitorix
```

**Allow the port 8080 used by the service:**
```bash
sudo ufw allow 8080
```

**Access Monitorix:**
```
http://YOUR_SERVER_IP:8080/monitorix
```

---
