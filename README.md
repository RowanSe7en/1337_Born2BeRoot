
```
██████╗  ██████╗ ██████╗ ███╗   ██╗██████╗ ██████╗ ███████╗██████╗  ██████╗  ██████╗ ████████╗
██╔══██╗██╔═══██╗██╔══██╗████╗  ██║╚════██╗██╔══██╗██╔════╝██╔══██╗██╔═══██╗██╔═══██╗╚══██╔══╝
██████╔╝██║   ██║██████╔╝██╔██╗ ██║ █████╔╝██████╔╝█████╗  ██████╔╝██║   ██║██║   ██║   ██║   
██╔══██╗██║   ██║██╔══██╗██║╚██╗██║██╔═══╝ ██╔══██╗██╔══╝  ██╔══██╗██║   ██║██║   ██║   ██║   
██████╔╝╚██████╔╝██║  ██║██║ ╚████║███████╗██████╔╝███████╗██║  ██║╚██████╔╝╚██████╔╝   ██║   
╚═════╝  ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═══╝╚══════╝╚═════╝ ╚══════╝╚═╝  ╚═╝ ╚═════╝  ╚═════╝    ╚═╝
```

*This project has been created as part of the 42 curriculum.*

---

# Born2beRoot 🖥️

## Description

**Born2beRoot** is a system administration project from the 42 curriculum that introduces students to the world of **virtualization** and **Linux server management**. The goal is to set up a fully functional, headless Linux server inside a virtual machine, following strict security and configuration rules. By the end of this project, you will have hands-on experience with partitioning, encryption, user management, firewalls, SSH, sudo policies, and system monitoring — all without a graphical interface.

### What you'll learn

- How to create and configure a virtual machine using **VirtualBox** (or UTM)
- How to install and configure **Debian** (or Rocky Linux) as a minimal server OS
- How to set up **encrypted LVM partitions**
- How to harden a Linux system: strong passwords, AppArmor/SELinux, firewall rules
- How to manage users, groups, and sudo privileges securely
- How to configure and restrict **SSH** access
- How to write a **bash monitoring script** and automate it with cron

---

## Project Requirements Summary

### Operating System Choice

You must choose between:
- **Debian** (latest stable — recommended for beginners)
- **Rocky Linux** (latest stable — enterprise-grade, more complex)

> ⚠️ No graphical server (X.org, Wayland, or equivalent) is allowed. Installing one results in a grade of **0**.

---

## Instructions

### 1. General Guidelines

- Use **VirtualBox** (mandatory) or UTM if VirtualBox is unavailable
- **Snapshots are forbidden** — any snapshot found during defense = grade 0
- You only submit a `signature.txt` file at the root of your Git repository, containing the SHA1 hash of your VM's virtual disk
- The VM itself must **never** be submitted to the repository

#### Getting the disk signature

**Linux:**
```bash
sha1sum ~/VirtualBox\ VMs/<your_vm_name>/<disk>.vdi
```
**macOS:**
```bash
shasum ~/VirtualBox\ VMs/<your_vm_name>/<disk>.vdi
```
**Windows:**
```cmd
certUtil -hashfile <disk>.vdi sha1
```
**Mac M1 (UTM):**
```bash
shasum ~/Library/Containers/com.utmapp.UTM/Data/Documents/<vm>.utm/Images/disk-0.qcow2
```

---

### 2. Disk Partitioning (Mandatory)

You must create **at least 2 encrypted partitions using LVM**.

#### Recommended layout (mandatory part):

```
Physical Disk (sda) — ~8.6 GB
├── /boot        (~500 MB, ext2/ext4, unencrypted)
└── sda5_crypt   (~8.1 GB, LUKS encrypted)
      ├── LV root   (~3 GB, ext4, mounted at /)
      ├── LV swap   (~1 GB, swap)
      └── LV home   (~4 GB, ext4, mounted at /home)
```

#### Bonus part layout:

```
Physical Disk (sda) — ~30.8 GB
├── /boot              (500 MB)
└── sda5_crypt         (30.3 GB, LUKS encrypted)
      ├── LVMGroup-root       (10 GB, /)
      ├── LVMGroup-swap       (2.3 GB, [SWAP])
      ├── LVMGroup-home       (5 GB, /home)
      ├── LVMGroup-var        (3 GB, /var)
      ├── LVMGroup-srv        (3 GB, /srv)
      ├── LVMGroup-tmp        (3 GB, /tmp)
      └── LVMGroup-var--log   (4 GB, /var/log)
```

#### Key concepts

| Concept | Description |
|---|---|
| **LVM** | Logical Volume Manager — allows flexible, resizable storage layers above physical partitions |
| **LUKS** | Linux Unified Key Setup — encrypts the entire partition; a passphrase is required at every boot |
| **PV** | Physical Volume — a raw disk or partition given to LVM |
| **VG** | Volume Group — a pool of storage built from one or more PVs |
| **LV** | Logical Volume — a flexible "partition" carved out of a VG |

> 💡 Disk sizes shown are examples. Resize LVs later with `sudo lvresize -r -L <size> /dev/<VG>/<LV>`

---

### 3. OS Configuration

#### AppArmor (Debian) / SELinux (Rocky)

AppArmor must be running at startup on Debian. Check with:

```bash
sudo systemctl status apparmor
sudo aa-status
```

SELinux must be running and in enforcing mode on Rocky:

```bash
sestatus
```

#### Hostname

Your VM hostname must be your 42 login followed by `42`:

```bash
sudo hostnamectl set-hostname <login>42
sudo nano /etc/hostname       # set <login>42
sudo nano /etc/hosts          # update 127.0.1.1 to <login>42
```

---

### 4. SSH Configuration

SSH must run on **port 4242** and **root login via SSH must be disabled**.

Install the SSH server:
```bash
sudo apt install openssh-server
```

Edit `/etc/ssh/sshd_config`:
```
Port 4242
PermitRootLogin no
```

Restart SSH:
```bash
sudo systemctl restart ssh
sudo systemctl enable ssh
```

Test locally:
```bash
ssh <username>@127.0.0.1 -p 4242
```

#### VirtualBox port forwarding (to connect from host machine)

Go to: **VM Settings → Network → Adapter 1 (NAT) → Advanced → Port Forwarding**

| Name | Protocol | Host IP | Host Port | Guest Port |
|---|---|---|---|---|
| SSH | TCP | 127.0.0.1 | 4242 | 4242 |

---

### 5. Firewall (UFW)

Install and configure UFW (Debian):

```bash
sudo apt install ufw
sudo ufw enable
sudo ufw allow 4242
sudo ufw status
```

The firewall must be **active at VM startup** and only **port 4242** must be open (in the mandatory part).

For Rocky Linux, use `firewalld` instead of UFW.

---

### 6. Strong Password Policy

#### `/etc/login.defs` — expiration settings

```bash
sudo nano /etc/login.defs
```

Set:
```
PASS_MAX_DAYS   30
PASS_MIN_DAYS   2
PASS_WARN_AGE   7
```

Apply to existing users (including root):
```bash
sudo chage -M 30 -m 2 -W 7 <username>
sudo chage -M 30 -m 2 -W 7 root
```

#### `/etc/pam.d/common-password` — complexity rules

Install the quality module:
```bash
sudo apt install libpam-pwquality
```

Edit `/etc/pam.d/common-password` and add/modify the `pam_pwquality.so` line:

```
password requisite pam_pwquality.so retry=3 minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```

| Option | Meaning |
|---|---|
| `minlen=10` | Minimum 10 characters |
| `ucredit=-1` | At least 1 uppercase letter |
| `lcredit=-1` | At least 1 lowercase letter |
| `dcredit=-1` | At least 1 digit |
| `maxrepeat=3` | No more than 3 consecutive identical characters |
| `reject_username` | Password cannot contain the username |
| `difok=7` | At least 7 characters different from the old password |
| `enforce_for_root` | Root must also comply |

> ⚠️ After setting up the policy, change all passwords (including root) to comply:
> ```bash
> sudo passwd <username>
> sudo passwd root
> ```

---

### 7. Sudo Configuration

Install sudo:
```bash
apt install sudo
```

Add your user to the sudo group:
```bash
sudo usermod -aG sudo <username>
# or
sudo adduser <username> sudo
```

Edit sudoers safely with:
```bash
sudo visudo
# or for a drop-in file:
sudo visudo -f /etc/sudoers.d/sudoers_rules
```

Required configuration in `/etc/sudoers` or `/etc/sudoers.d/`:

```
Defaults    passwd_tries=3
Defaults    badpass_message="Wrong password. You have 3 attempts."
Defaults    log_input, log_output
Defaults    iolog_dir="/var/log/sudo"
Defaults    requiretty
Defaults    secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```

Create the log directory:
```bash
sudo mkdir -p /var/log/sudo
```

| Rule | Purpose |
|---|---|
| `passwd_tries=3` | Limit sudo attempts to 3 |
| `badpass_message` | Custom error message on wrong password |
| `log_input, log_output` | Log all sudo I/O sessions |
| `iolog_dir` | Directory where sudo logs are saved |
| `requiretty` | Sudo only works in real terminal sessions |
| `secure_path` | Restrict PATH used by sudo to trusted directories |

---

### 8. Users & Groups

```bash
# Create your main user (if not already created during installation)
sudo adduser <login>

# Create the user42 group
sudo groupadd user42

# Add your user to user42 and sudo groups
sudo usermod -aG user42,sudo <login>

# Verify
groups <login>
getent group user42
getent group sudo
```

#### During evaluation — creating a new user and group

```bash
sudo adduser <new_user>
sudo addgroup evaluating
sudo adduser <new_user> evaluating
groups <new_user>
```

---

### 9. Monitoring Script

Create `/usr/local/bin/monitoring.sh` (or `/etc/monitoring/monitoring.sh`):

```bash
#!/bin/bash

# OS architecture and kernel version
arch=$(uname -a)

# Physical CPUs
cpuf=$(grep "physical id" /proc/cpuinfo | uniq | wc -l)

# Virtual CPUs
cpuv=$(nproc)

# RAM usage
ram_total=$(free --mega | grep Mem | awk '{print $2}')
ram_use=$(free --mega | grep Mem | awk '{print $3}')
ram_percent=$(free --mega | grep Mem | awk '{printf("%.2f", $3/$2*100)}')

# Disk usage
disk_total=$(df -m | grep -E '^/dev/' | grep -v "/boot" | awk '{disk_t += $2} END {printf("%.1f", disk_t/1024)}')
disk_use=$(df -m | grep "^/dev/" | grep -v "/boot" | awk '{disk_u += $3} END {print disk_u}')
disk_percent=$(df -m | grep "^/dev/" | grep -v "/boot" | awk '{disk_u += $3; disk_t += $2} END {printf("%.2f", disk_u/disk_t*100)}')

# CPU load
cpu_fin=$(vmstat 1 2 | tail -1 | awk '{printf "%.1f", 100 - $15}')

# Last boot
lb=$(who -b | awk '{print $3, $4}')

# LVM active?
lvmu=$(lsblk | grep -q "lvm" && echo "Yes" || echo "No")

# Active TCP connections
tcpc=$(ss -ta | grep ESTAB | wc -l)

# Logged-in users
ulog=$(who | awk '{print $1}' | sort -u | wc -l)

# Network info
ip=$(hostname -I)
mac=$(ip link | grep "link/ether" | awk '{print $2}')

# Sudo command count
cmnd=$(journalctl -q _COMM=sudo | grep COMMAND | wc -l)

wall "  #Architecture: $arch
  #CPU physical: $cpuf
  #vCPU: $cpuv
  #Memory Usage: $ram_use/${ram_total}MB ($ram_percent%)
  #Disk Usage: $disk_use/${disk_total}GB ($disk_percent%)
  #CPU load: $cpu_fin%
  #Last boot: $lb
  #LVM use: $lvmu
  #Connections TCP: $tcpc ESTABLISHED
  #User log: $ulog
  #Network: IP $ip ($mac)
  #Sudo: $cmnd cmd"
```

Make it executable:
```bash
sudo chmod +x /usr/local/bin/monitoring.sh
```

#### Schedule with cron (every 10 minutes)

```bash
sudo crontab -e
```

Add:
```
*/10 * * * * /usr/local/bin/monitoring.sh
```

> 💡 `*/10` means "every 10 minutes". `10` alone would mean "at minute 10 of every hour" — **not the same thing**.

To interrupt it without modifying the script, comment out or remove the cron entry during evaluation.

#### Cron syntax reference

```
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of week (0–7, 0 and 7 = Sunday)
│ │ │ └──── Month (1–12)
│ │ └────── Day of month (1–31)
│ └──────── Hour (0–23)
└────────── Minute (0–59)
```
---

## Submission

```
your_git_repo/
└── signature.txt   ← SHA1 hash of your .vdi or .qcow2 file
```

> ⚠️ Do **not** add your VM to the repository. If the signature in `signature.txt` doesn't match your VM's disk at evaluation time, your grade is 0.

---

## Project Description & Technical Comparisons

### Why Debian?

Debian was chosen for this project because it provides a **stable, minimal, and well-documented** environment that is ideal for learning core Linux administration without the distraction of enterprise-specific tooling.

| | Debian | Rocky Linux |
|---|---|---|
| **Base** | Debian upstream | RHEL-compatible |
| **Package manager** | APT (`apt`, `aptitude`) | DNF/YUM (`dnf`) |
| **Security module** | AppArmor | SELinux |
| **Firewall** | UFW | firewalld |
| **Complexity** | Beginner-friendly | More complex |
| **Default shell** | bash | bash |
| **Use case** | Servers, desktops | Enterprise servers |
| **Recommended for B2R** | ✅ Yes | Possible but harder |

### AppArmor vs SELinux

| | AppArmor | SELinux |
|---|---|---|
| **Model** | Profile-based (per program) | Label-based (entire system) |
| **Complexity** | Simpler | More complex |
| **Configuration** | `/etc/apparmor.d/` | `/etc/selinux/` |
| **Default distro** | Debian, Ubuntu | RHEL, Rocky, Fedora |
| **Modes** | Enforce / Complain | Enforcing / Permissive / Disabled |
| **Granularity** | Per-application | System-wide |

### UFW vs firewalld

| | UFW | firewalld |
|---|---|---|
| **Full name** | Uncomplicated Firewall | Dynamic Firewall |
| **Backend** | iptables/nftables | nftables/iptables |
| **Interface** | Simple CLI | Zones-based CLI + GUI |
| **Default distro** | Debian, Ubuntu | RHEL, Rocky, Fedora |
| **Configuration style** | Simple rules | Zones and services |
| **Ease of use** | Very easy | Moderate |

### VirtualBox vs UTM

| | VirtualBox | UTM |
|---|---|---|
| **Type** | Type-2 hypervisor | Type-2 hypervisor (QEMU-based) |
| **Platform** | Windows, Linux, macOS | macOS only |
| **Apple Silicon support** | Limited (x86 only) | Native ARM64 |
| **Free** | Yes | Yes |
| **Use in B2R** | Mandatory (default) | Alternative on Mac M1/M2 |

---

## Key Commands Reference

### Users & Groups

```bash
adduser <username>                      # Create a new user (interactive)
userdel -r <username>                   # Delete user and home directory
usermod -aG <group> <username>          # Add user to group (non-destructive)
adduser <username> <group>              # Alternative: add user to group
deluser <username> <group>              # Remove user from group
groups <username>                       # List user's groups
getent passwd <username>               # Check user exists
getent group <group>                   # List group members
groupadd <group>                        # Create a group
groupdel <group>                        # Delete a group
chage -l <username>                     # View password aging info
chage -M 30 -m 2 -W 7 <username>       # Set password aging policy
```

### SSH

```bash
sudo systemctl status ssh
sudo systemctl enable ssh
sudo systemctl restart ssh
ssh <user>@127.0.0.1 -p 4242
```

### UFW

```bash
sudo ufw status
sudo ufw enable / disable
sudo ufw allow 4242
sudo ufw deny 80
sudo ufw reload
```

### Sudo

```bash
sudo visudo                             # Safely edit sudoers
sudo visudo -f /etc/sudoers.d/rules    # Edit a drop-in file
```

### LVM

```bash
lsblk                                  # Display block devices
pvdisplay / pvscan                     # Show physical volumes
vgdisplay / vgscan                     # Show volume groups
lvdisplay / lvscan                     # Show logical volumes
sudo lvresize -r -L 5G /dev/<VG>/<LV> # Resize a logical volume
```

### Monitoring

```bash
uname -a                               # OS architecture and kernel
lscpu                                  # CPU info
free --mega                            # RAM usage
df -m                                  # Disk usage
vmstat 1 2                             # CPU load
who -b                                 # Last boot time
ss -ta                                 # TCP connections
journalctl -q _COMM=sudo | grep COMMAND | wc -l   # Sudo commands count
wall "message"                         # Broadcast to all terminals
crontab -e                             # Edit cron jobs
sudo crontab -l                        # List root cron jobs
```

### AppArmor

```bash
sudo systemctl status apparmor
sudo systemctl enable apparmor
sudo aa-status
```
---

## Resources

### Official Documentation

- [Debian Official Documentation](https://www.debian.org/doc/)
- [Debian Reference Manual](https://www.debian.org/doc/manuals/debian-reference/)
- [LVM HowTo — tldp.org](https://tldp.org/HOWTO/LVM-HOWTO/)
- [UFW Documentation — Ubuntu](https://help.ubuntu.com/community/UFW)
- [OpenSSH Manual Pages](https://www.openssh.com/manual.html)
- [AppArmor Wiki](https://wiki.ubuntu.com/AppArmor)
- [PAM Configuration Guide](https://linux.die.net/man/5/pam.conf)
- [sudoers(5) Man Page](https://www.sudo.ws/docs/man/sudoers.man/)
- [Cron and Crontab — man7.org](https://man7.org/linux/man-pages/man5/crontab.5.html)
- [Lighttpd Documentation](https://redmine.lighttpd.net/projects/lighttpd/wiki/Docs)
- [MariaDB Documentation](https://mariadb.com/kb/en/documentation/)
- [WordPress Installation Guide](https://wordpress.org/documentation/article/how-to-install-wordpress/)

### Articles & Tutorials

- [42 Born2beRoot Walkthrough (community guides)](https://github.com/topics/born2beroot)
- [Linux Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)
- [Understanding LVM — Red Hat](https://www.redhat.com/sysadmin/lvm-vs-partitioning)
- [Crontab Guru — visual cron expression editor](https://crontab.guru/)

### AI Usage

AI was used in this project in the following ways:

- **Concept clarification**: Understanding low-level concepts such as how LUKS encryption interacts with LVM, or how PAM modules are chained in `/etc/pam.d/common-password`.
- **Command syntax reference**: Verifying exact flags for commands like `chage`, `usermod`, `lvresize`, and `visudo`.
- **Script debugging**: Checking bash logic for edge cases in `monitoring.sh`, such as counting unique logged-in users or computing CPU load with `vmstat`.

> ⚠️ Per 42 AI policy: AI was used as a learning aid to understand concepts, not to generate solutions blindly. All configurations were manually tested and understood before being applied.
