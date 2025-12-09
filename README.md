
# Born2beRoot – System Administration Project

> This project has been created as part of the 42 curriculum by **achahi**

## 📌 Description

**Born2beRoot** is a system administration project designed to introduce you to Linux server configuration, virtualization, secure partitioning with LVM encryption, firewall rules, SSH setup, user management, password policies, and system monitoring through automation.

The project involves installing a secure virtual server using Debian (or Rocky) and configuring it following strict 42 security rules.

This README explains all the concepts used, why I chose Debian, how each component works, and how the system was configured.

----------

## 📑 Table of Contents

-   [Instructions](#-instructions-how-to-userun-the-project)
-   [Why I Chose Debian](#-why-i-chose-debian)
-   [Virtualization](#-virtualization)
-   [Hypervisors](#-hypervisors)
-   [LVM and Disk Partitioning](#-lvm-and-disk-partitioning)
-   [Server](#-server)
-   [APT and Aptitude](#-apt-and-aptitude)
-   [SSH](#-ssh)
-   [Hostname](#-hostname)
-   [AppArmor Security](#-apparmor-security)
-   [Firewall](#-firewall)
-   [Password Policies](#-password-policies)
-   [Sudo Configuration](#-sudo-configuration)
-   [Monitoring Script](#-monitoring-script)
-   [Bonus: WordPress Setup](#-bonus-wordpress-setup-lighttpd--mariadb--php)
-   [Resources](#-resources)
-   [How AI Was Used](#-how-ai-was-used)

----------

## 📌 Instructions (How to use/run the project)

### 🔧 1. Start the Virtual Machine

Boot the VM through VirtualBox (or UTM if on M1/M2).

### 🔑 2. Connect via SSH

```bash
ssh -p 4242 youruser@server_ip

```

### 🖥 3. Run the Monitoring Script

```bash
sudo /usr/local/bin/monitoring.sh

```

### 🔁 4. Cron runs it automatically

The script broadcasts information every 10 minutes.

### 📄 5. To view sudo logs

```bash
sudo cat /var/log/sudo/sudo.log

```

----------

## 🟦 Why I Chose Debian

Debian is **stable**, **secure**, **beginner-friendly**, and widely used in servers. Rocky Linux uses SELinux, which is more complex for newcomers. Since the project focuses on basics, Debian was the most efficient and clear choice.

### Debian vs Rocky Linux

Feature

Debian

Rocky Linux

**Security module**

AppArmor

SELinux

**Difficulty**

Easier

More complex

**Package manager**

APT

DNF

**Stability**

Very stable

Enterprise-level stable

**Recommended for beginners**

✔️

❌

----------

## 🟦 Virtualization

**Virtualization** allows running multiple independent operating systems on one physical computer.

A **Virtual Machine (VM)** is a simulated computer created by software.

----------

## 🟦 Hypervisors

### Type 1 — Bare Metal

Runs directly on hardware (ESXi, Hyper-V Server).

### Type 2 — Hosted

Runs on top of an existing OS (VirtualBox, VMware, UTM).

### VirtualBox vs UTM

Feature

VirtualBox

UTM

**Platform**

All systems

macOS ARM only

**Speed**

Faster

Slower

**Suitable for Born2beRoot**

✔️

Only for M1/M2

----------

## 🟦 LVM and Disk Partitioning

### What is LVM?

**LVM (Logical Volume Manager)** is a flexible method for managing storage. It allows resizing, combining disks, snapshots, and encrypted volumes.

### Why Use LVM?

-   **Resize partitions easily**
-   **Increase security with encryption**
-   **Organize server directories**
-   **Improve flexibility over fixed partitions**

### Partition Structure Used

Volume

Mount

Purpose

`/boot`

Primary

Holds kernel and bootloader

`sda5`

LUKS

Encrypted container

`LVMGroup-root`

`/`

System files

`LVMGroup-swap`

`swap`

Virtual memory

`LVMGroup-home`

`/home`

User files

`LVMGroup-var`

`/var`

Variable data

`LVMGroup-srv`

`/srv`

Service data

`LVMGroup-tmp`

`/tmp`

Temporary files

`LVMGroup-var--log`

`/var/log`

System logs

### LVM Architecture (3 Levels)

-   **PV** – Physical Volume → your encrypted partition
-   **VG** – Volume Group → storage pool
-   **LV** – Logical Volumes → usable partitions

----------

## 🟦 Server

A **server** is a machine that provides services to clients: SSH, web hosting, database, DNS, etc.

----------

## 🟦 APT and Aptitude

### APT

Basic package manager (install, update, remove packages).

### Aptitude

Advanced package manager with conflict-resolution suggestions.

----------

## 🟦 SSH

**SSH** is a secure protocol for remote terminal access.

### Configuration required:

-   Port changed from `22` → `4242`
-   `PermitRootLogin no`
-   SSH enabled and active

```bash
sudo nano /etc/ssh/sshd_config

```

Change:

```
Port 4242
PermitRootLogin no

```

Restart SSH:

```bash
sudo systemctl restart ssh

```

----------

## 🟦 Hostname

Used to identify the machine in a network.

```bash
sudo hostnamectl set-hostname yourlogin42

```

----------

## 🟦 AppArmor Security

**AppArmor** restricts application privileges based on profiles.

### AppArmor vs SELinux

Feature

AppArmor

SELinux

**Complexity**

Easy

Hard

**Rules**

Path-based

Label-based

**OS**

Debian

Rocky

**Recommended for beginners**

✔️

❌

Check status:

```bash
sudo aa-status

```

----------

## 🟦 Firewall

### UFW (for Debian)

Simple interface to Netfilter firewall.

```bash
sudo ufw allow 4242
sudo ufw enable

```

### UFW vs firewalld

Feature

UFW

firewalld

**Difficulty**

Easy

Medium

**Style**

Static rules

Dynamic zones

**OS**

Debian

Rocky

----------

## 🟦 Password Policies

### login.defs

Edit `/etc/login.defs`:

```bash
PASS_MAX_DAYS 30
PASS_MIN_DAYS 2
PASS_WARN_AGE 7

```

### PAM password rules (libpam-pwquality)

Install:

```bash
sudo apt install libpam-pwquality

```

Edit `/etc/pam.d/common-password`:

```
password requisite pam_pwquality.so retry=3 minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 reject_username enforce_for_root difok=7

```

**Requirements:**

-   Min length: **10**
-   **1 uppercase**
-   **1 lowercase**
-   **1 number**
-   **No username**
-   **No repeated characters**
-   **7 different chars vs previous password**

----------

## 🟦 Sudo Configuration

### Requirements:

-   **3 tries max**
-   **Custom wrong password message**
-   **Logging inputs/outputs**
-   **TTY required**
-   **Secure PATH**
-   **Password required every time** (`timestamp_timeout=0`)

### Configuration

Edit with:

```bash
sudo visudo

```

Add these lines:

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

### Logs stored in:

```bash
/var/log/sudo/

```

Create the directory:

```bash
sudo mkdir -p /var/log/sudo

```

----------

## 🟦 Monitoring Script

A Bash script that broadcasts system information at startup and every 10 minutes using `wall`.

### Displays:

-   Architecture
-   CPU physical & virtual
-   RAM usage
-   Disk usage
-   CPU load
-   Last boot
-   LVM use
-   TCP connections
-   Logged-in users
-   IP & MAC address
-   Sudo command count

### Script Location

Create the script:

```bash
sudo vim /usr/local/bin/monitoring.sh

```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/monitoring.sh

```

### Cron setup:

Edit root's crontab:

```bash
sudo crontab -e

```

Add this line:

```bash
*/10 * * * * /usr/local/bin/monitoring.sh

```

This runs the script every 10 minutes.

----------

## 🟦 Bonus: WordPress Setup (Lighttpd + MariaDB + PHP)

### Includes:

-   **Web server** (lighttpd)
-   **Firewall rule** (port 80)
-   **MariaDB database**
-   **PHP configuration**
-   **WordPress extraction & setup**
-   **FastCGI module**

### Installation Steps:

1.  **Install web server:**

```bash
sudo apt install lighttpd

```

2.  **Allow HTTP traffic:**

```bash
sudo ufw allow 80

```

3.  **Install database:**

```bash
sudo apt install mariadb-server

```

4.  **Secure database:**

```bash
sudo mysql_secure_installation

```

5.  **Create WordPress database:**

```bash
sudo mariadb
CREATE DATABASE my_wordpress;
GRANT ALL ON my_wordpress.* TO 'wpuser'@'localhost' IDENTIFIED BY 'YourPassword123!' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit;

```

6.  **Install PHP:**

```bash
sudo apt install php-cgi php-mysql

```

7.  **Download WordPress:**

```bash
cd /var/www/
sudo wget http://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz
sudo rm latest.tar.gz
sudo mv html/ html_back
sudo mv wordpress/ html
sudo chmod -R 755 html

```

8.  **Configure WordPress:**

```bash
cd /var/www/html
sudo cp wp-config-sample.php wp-config.php
sudo vim wp-config.php

```

Update these lines with your database info:

```php
define( 'DB_NAME', 'my_wordpress' );
define( 'DB_USER', 'wpuser' );
define( 'DB_PASSWORD', 'YourPassword123!' );

```

9.  **Enable FastCGI:**

```bash
sudo lighty-enable-mod fastcgi-php
sudo service lighttpd force-reload

```

10.  **Access WordPress:**

```
http://YOUR_SERVER_IP

```

### Optional extra service: **Monitorix**

```bash
sudo apt install monitorix -y
sudo systemctl restart monitorix
sudo ufw allow 8080

```

Access at: `http://YOUR_SERVER_IP:8080/monitorix`

----------

## 📚 Resources

-   [Debian Documentation](https://www.debian.org/doc/)
-   [VirtualBox Docs](https://www.virtualbox.org/wiki/Documentation)
-   [Linux Manual Pages](https://man7.org/linux/man-pages/)
-   [42 Intranet Articles](https://intra.42.fr/)
-   PAM / SSH / UFW guides
-   LVM documentation

----------

## 🤖 How AI Was Used

AI was used **only** to:

-   Help restructure explanations
-   Improve readability
-   Rewrite the README format

**No configuration files, commands, or implementation were produced by AI.**

----------

## 📝 Notes

> ✅ All configurations have been tested and verified  
> ✅ Follows 42 Born2beRoot subject requirements  
> ✅ Includes both mandatory and bonus parts

----------

**Made with  by achahi for the 42 Network**
