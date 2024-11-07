---
title: "Linux System Administration Essentials (Part 2): Disk Partitioning, User Management and Network Configuration"
date: 2024-11-07 13:00:00 +0800
categories: [Cyber Security, Linux System Administration]
tags: [linux, system administration, sysadmin, file partitioning, linux basics, linux file system, user and group management, network configurations, kundan dhupkar]
author: KD
comments: true
image: "assets/img/diagrams/writeup_three/title.png"
toc: true
description: In this part, we dive into partitioning and setting up the file system. You’ll also learn how to effectively manage users and groups, configure network settings and troubleshoot network issues. 
---

## Introduction

In the first part of this series, we covered foundational Linux concepts, including the OS, kernel and file systems. If you haven’t checked that out yet, you can read it [here](https://securitymates.github.io/posts/Linux-System-Administration-Essentials-(Part-1)-Introduction/). In this part, we’ll dive deeper into three essential system administration tasks:

1. Setting up disk partitions and file systems
2. Managing users and groups
3. Configuring and troubleshooting network settings

Let’s explore each of these in detail!

---

## Setting Up Disk Partitions and File Systems

Before creating partitions, let’s understand what partitioning is and why it's important.

Partitioning is the process of logically dividing a hard drive (or SSD) into separate sections, called "partitions". Each partition can function as an independent drive, which helps us organize data, install multiple operating systems or keep certain types of files separately.

**Reasons to Partition a Disk:**
1. **Organization**: Separating system files, user data and backups.
2. **Multiple OS Support**: Install different operating systems (e.g. Windows and Linux) on the same disk.
3. **Performance**: Some workloads benefit from dedicated partitions.
4. **Security**: Certain partitions can be restricted to enhance security.

**Types of Partitions:**
1. **Primary Partitions**: Up to four primary partitions can be created on a disk.
2. **Extended Partitions**: To create more than four partitions, an extended partition can be created to hold multiple logical partitions.
3. **Logical Partitions**: These reside within an extended partition, allowing more than four partitions on a disk.

**Why Only Four Primary Partitions?**

The MBR(Master Boot Record) is stored on the disk's first sector. Each sector of hard drive is of standard size 512 bytes so size of MBR is 512 bytes as well.
In MBR, the first 446 bytes contains boot code then 64 bytes contain partition table and remaining 2 bytes contain boot signature.
Each entry of the partition takes 16 bytes that's why you can only create 4 primary partitions(64 / 16 = 4).

For more than 4 partitions, we use extended partitions with logical partitions. In extended partition there is again first sector of 512 bytes which contains EBR (Extended Boot Record) but this doesnt contain boot code so 446 is free then there is again 64 bytes partition table like MBR but it only uses 32 bytes leaving 32 bytes unused. In that first 16 bytes describe the first logical drive and the remaining 16 bytes are pointer to next logical drive. Similarly, this chain continues.

![Layout in MBR vs GPT](assets/img/diagrams/writeup_three/mbr_gpt.png)

_Layout in MBR vs GPT from simplylinuxfaq_

Newer systems now use the GUID Partition Table (GPT), which supports a much higher number of partitions.

Each partition can be formatted with a file system, which determines how data is stored and retrieved. Common Linux file systems include:

1.**ext4**: Default for many Linux distributions, supports large files and journaling.
2.**xfs**: Good for high-performance tasks.
3.**btrfs**: Offers advanced features like snapshots and dynamic resizing.


**Creating Partitions:**

To set up a partition and file system, follow these steps:

1. Open a terminal and list disks with:
   ```bash
   sudo fdisk -l
   ```
   Alternatively, you can also see this using the GUI Disk Utility.

![List Disks](assets/img/diagrams/writeup_three/list_disks.png)

_List Disks_

2. Use `fdisk` to create a partition:
   ```bash
   sudo fdisk /dev/sdX (replace X with your drive letter)
   ```
   - Type `n` to create a new partition.
   - Choose `p` for primary or `l` for logical.
   - Set the partition number and size.
   - Type `w` to write changes.

![Creating Partition](assets/img/diagrams/writeup_three/disk_cmd.png)

_Creating Partition_

3. Reboot to apply changes. Then format the partition:
   ```bash
   sudo mkfs.ext4 /dev/sdX (replace X with your drive letter)
   ```
![Formatting with Filesystem](assets/img/diagrams/writeup_three/disk_cmd2.png)

_Formatting with Filesystem_

4. To make the partition persistent, add it to `/etc/fstab`:
The format to add entry in /etc/fstab is:
   ```bash
   UUID=[UUID] /mountpoint ext4 defaults 0 0
   ```
   You can get the UUID with `blkid` command.

![/etc/fstab entry](assets/img/diagrams/writeup_three/disk_cmd3.png)

_Adding entry to /etc/fstab_

You can learn more about /etc/fstab file in detail from this article : [https://linuxconfig.org/how-fstab-works-introduction-to-the-etc-fstab-file-on-linux](https://linuxconfig.org/how-fstab-works-introduction-to-the-etc-fstab-file-on-linux)

5. Finally, mount the partition:
   ```bash
   sudo mount /dev/sdXn /mnt
   # sudo mount /dev/sda1 /mnt (example)
   ```

Partitioning is a fundamental skill for efficient storage management in Linux. With these commands and concepts, you’re well-equipped to manage partitions effectively.

---

## Managing Users and Groups on a Linux System

Managing users and groups is essential in Linux administration, allowing control over system access and permissions. Here’s a comprehensive overview, from the fundamentals to more advanced concepts.


### 1. **Basic Concepts**

- **Users**: A user is an individual account that can log into the system. Each user has a unique username and associated permissions.
- **Root User**: The superuser (root) has unrestricted access to all commands and files on the system. It’s used for administrative tasks.
- **Groups**: Collections of users with similar access needs. Groups simplify permission management across multiple users.

### 2. **User and Group Management Commands**

**User Management**:

- **Adding a User**:
  ```bash
  sudo adduser username
  ```
  This creates a new user and prompts you to set a password and other details.

![adduser command](assets/img/diagrams/writeup_three/adduser.png)

_Adding user 'securitymates'_

- **Deleting a User**:
  ```bash
  sudo deluser username
  ```
  This removes the user account.

![deluser command](assets/img/diagrams/writeup_three/deluser.png)

_Deleting user 'securitymates'_

- **Modifying a User**:
  ```bash
  sudo usermod -aG groupname username
  ```
  This command adds a user to a group (`-aG` means append to groups).

- **Viewing User Information**:
  ```bash
  id username
  ```
  This shows the user’s UID (user ID), GID (group ID), and group memberships.


**Group Management**:
- **Adding a Group**:
  ```bash
  sudo addgroup groupname
  ```
  This creates a new group.

![addgroup command](assets/img/diagrams/writeup_three/addgrp.png)

_Creating new group 'securitymatesblog'_

- **Deleting a Group**:
  ```bash
  sudo delgroup groupname
  ```
  This removes the group.

- **Modifying a Group**:
  ```bash
  sudo groupmod -n newgroupname oldgroupname
  ```
  This changes the name of an existing group.

- **Viewing Group Information**:
  ```bash
  getent group groupname
  ```
  This displays information about the specified group.


### 3. **Understanding /etc/passwd and /etc/group Files**

- **/etc/passwd**: This file contains information about user accounts. Each line corresponds to a user and includes fields like:
  - Username
  - Password (usually an 'x' indicating the password is stored in /etc/shadow)
  - User ID (UID)
  - Group ID (GID)
  - User info (e.g., full name)
  - Home directory
  - Default shell

![/etc/passwd file](assets/img/diagrams/writeup_three/etc_passwd.png)

_/etc/passwd file_

- **/etc/group**: This file contains information about groups. Each line includes:
  - Group name
  - Password (usually not used)
  - GID
  - List of users in the group


### 4. **Advanced User and Group Management**

#### Password Management
- **Changing a User’s Password**:
  ```bash
  sudo passwd username
  ```
  This prompts you to enter a new password for the user.

- **Password Aging**:
  You can set password aging policies using `chage` to control when users must change their passwords and how long they’re valid.

#### Access Control Lists (ACLs)
- **Definition**: ACLs allow for more granular permission settings than traditional Unix permissions.
- **Setting an ACL**:
  ```bash
  setfacl -m u:username:rwx /path/to/file
  ```
  This grants the specified user read, write, and execute permissions on a file.

- **Viewing ACLs**:
  ```bash
  getfacl /path/to/file
  ```

![Access Control Lists(ACLs)](assets/img/diagrams/writeup_three/acl.png)

_Setting ACLs_

#### Managing System Users
- **System Users**: These are users that are created for specific services or applications (e.g., `www-data` for web servers). They typically have no login privileges.
- **Creating System Users**:
  ```bash
  sudo adduser --system --no-create-home username
  ```

By understanding and using these commands, user and group management on Linux becomes more secure and efficient.

---

## Network Configuration and Troubleshooting

### Configuring Network Interfaces
Ubuntu uses `netplan` for network configuration. The configuration file is typically located at `/etc/netplan/01-netcfg.yaml`.

#### Example Netplan Configuration
Edit the netplan config file:
```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```
Apply the changes:
```bash
sudo netplan apply
```

### Testing Network Connectivity
Use the following commands to troubleshoot network connectivity:

1. **Ping**: Checks connectivity to a host.
   ```bash
   ping -c 4 google.com
   ```

2. **Traceroute**: Diagnoses routing issues.
   ```bash
   traceroute google.com
   ```

3. **Checking Open Ports**: Use `ss` or `netstat` to display open ports:
   ```bash
   sudo ss -tuln
   ```

4. **Viewing Network Configuration**: Use `ip` to view network details:
   ```bash
   ip addr show
   ip route show
   ```

### Firewall Configuration with UFW
Ubuntu uses **UFW** (Uncomplicated Firewall) to manage firewall rules.

#### Basic UFW Commands
1. Enable UFW:
   ```bash
   sudo ufw enable
   ```
2. Allow SSH connections:
   ```bash
   sudo ufw allow ssh
   ```
3. Check firewall status:
   ```bash
   sudo ufw status
   ```

### DNS Troubleshooting
1. **Check DNS Resolution**:
   ```bash
   nslookup google.com
   ```
2. **Flush DNS Cache**:
   ```bash
   sudo systemd-resolve --flush-caches
   ```

---

## Final Thoughts

Disk partitioning, user management and network configuration are foundational Linux administration skills. By mastering these areas you’ll be prepared to handle essential tasks in managing Linux systems effectively and securely. 

If you have any specific questions or need more details on any part, feel free to ask! Stay tuned for the next part of this series! 

--- 

{% include comment.html %}
