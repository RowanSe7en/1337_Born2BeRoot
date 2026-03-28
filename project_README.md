*This project has been created as part of the 42 curriculum by <brouane>.*

# Born2beRoot

## Description

Born2beRoot is a system administration project from the 42 curriculum focused on setting up and securing a Linux server from scratch inside a virtual machine. The goal of this project is to understand the fundamentals of operating systems, user and permission management, services, networking, security policies, and automation. The server is configured to follow strict security rules while remaining functional and accessible through SSH for remote administration. This project emphasizes best practices in system hardening, service configuration, and monitoring, preparing students for real-world Linux server management scenarios.

---

## Project Description and Technical Choices

### Operating System Choice: Debian

For this project, **Debian** was chosen as the operating system.

**Pros of Debian:**
- Very stable and well-tested
- Large community and extensive documentation
- Lightweight and well-suited for servers
- Uses AppArmor by default, which is simpler to manage for beginners

**Cons of Debian:**
- Packages may not always be the most recent versions
- Slower release cycle compared to some distributions

**Debian vs Rocky Linux:**
- Debian focuses on stability and simplicity, while Rocky Linux aims for enterprise-level compatibility with Red Hat.
- Rocky Linux uses SELinux by default, which is more powerful but also more complex to configure.
- Debian is generally easier to manage for learning system administration fundamentals.

---

### Security Modules: AppArmor vs SELinux

- **AppArmor** (chosen): Easier to configure, profile-based, and sufficient for this project’s security requirements.
- **SELinux**: More granular and powerful but significantly more complex and error-prone for beginners.

---

### Firewall: UFW vs firewalld

- **UFW** (chosen): Simple, user-friendly, and well-integrated with Debian.
- **firewalld**: More dynamic and flexible, commonly used in Red Hat–based systems like Rocky Linux.

---

### Virtualization: VirtualBox vs UTM

- **VirtualBox** (chosen): Cross-platform, widely used, well-documented, and fully compatible with the project requirements.
- **UTM**: Better suited for macOS (especially Apple Silicon) but less commonly used in Linux-focused environments.

---

### Main Design Choices

- **Partitioning**: Separate logical volumes using LVM to improve flexibility and disk management.
- **Security policies**:
  - Strong password rules
  - Limited sudo privileges
  - Firewall enabled with only necessary ports open
- **User management**:
  - No direct root SSH login
  - Dedicated user with sudo access
- **Installed services**:
  - SSH for remote access
  - UFW for ports management

---

## Instructions

### Requirements
- VirtualBox
- Debian ISO
- A Linux or macOS host system

### Installation and Setup
1. Create a virtual machine in VirtualBox.
2. Install Debian with minimal packages.
3. Configure users, sudo, and password policies.
4. Set up SSH, firewall (UFW), and required services.
5. Configure system monitoring and cron jobs as required by the subject.

### Execution
- Connect to the server via SSH:
  ssh username@127.0.0.1 -p <port>

## Resources

### References and Documentation
- Born2beRoot GitBook: https://noreply.gitbook.io/born2beroot
- Debian ISO Download: https://www.debian.org/distrib/
- YouTube Tutorials:
  - Born2beRoot (Mandatory Part): Server Configuration | Linux Server | 42cursus Projects: https://www.youtube.com/watch?v=3Vw0HlJHLTQ
  - SSH Server Configuration and Security: https://www.youtube.com/watch?v=5JvLV2-ngCI&pp=ygUDc3No0gcJCU8KAYcqIYzv
  - UFW Firewall Basics on Debian: https://www.youtube.com/watch?v=XtRXm4FFK7Q&pp=ygUDdWZ3
  - Cron and Scheduled Tasks on Linux: https://www.youtube.com/watch?v=7cbP7fzn0D8&pp=ygUHY3JvbnRhYg%3D%3D

### AI Usage Disclosure
AI tools were used as a learning and documentation aid throughout this project. They helped clarify system administration concepts, explain Linux commands and security mechanisms (such as sudo, AppArmor, cron, and UFW), and structure written explanations in the README.
