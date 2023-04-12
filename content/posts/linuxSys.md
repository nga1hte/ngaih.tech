---
layout: post
title: "A deep dive into Linux" 
slug: "linuxsys"
date: "2023-04-10"
comments: false
categories:
  - linux
tags:
  - linux
  - system admin
---

Linux is an open-source operating system that is based on Unix. It was created by Linus Torvalds in 1991 and has since become one of the most widely used operating systems in the world. Linux is known for its stability, security, and flexibility, making it an ideal choice for a wide range of applications, from personal computers to enterprise-level servers.

One of the key features of Linux is that it is open source, which means that its source code is freely available for anyone to access, modify, and distribute. This has led to a large community of developers who contribute to the development of the operating system and create a wide range of software applications that run on it.

## Firmware

There are two main types of firmware that is used to boot up a computer.
- BIOS (Basic Input/Output System)
- UEFI (Unified Extensible Firmware Interface)

```BIOS``` is an older firmware and is stored on the computer's motherboard and its responsibility is to initialise and perform a hardware check of the system, like storage, memory and other devices. When the computer is powered on, the BIOS performs a Power-On Self Test (POST) to ensure that the hardware is functioning correctly. It also loads the OS on bootup. Eventhough BIOS is simple and reliable, it lacks some additional features which is addressed by ```UEFI```.

```UEFI``` is a newer type of firmware that boots up modern computers. It is more complex than BIOS and supports larger storage, secure boot and faster loading time. It also allows the development of custom boot loaders and other system software.

Eventhough BIOS has become outdated in current computing scenarios, most laptops and motherboards still comes with support for both firmwares.


## GRUB

GRUB (Grand Unified Bootloader) is a popular bootloader used on many Linux-based operating systems. It is responsible for loading the operating system kernel and other system files into memory during the boot process.

GRUB also supports menu-based interface, which allows users to select the operating system they want to boot and modify various boot infos. GRUB also has a CLI that can perform various maintenance tasks, such as repairing damaged boot files. 

GRUB has two versions:
- GRUB (/boot/grub/grub.conf)
- GRUB2 (/boot/grub/grub.cfg)

### Boot Methods

- BIOS/MBR: This is the traditional boot method used in older generation of computers. It uses BIOS firmware to boot the computer and ```MBR``` (Master Boot Record) to load the bootloader.
- UEFI/GPT: This is a newer boot method, it uses UEFI firmware to boot up the computer and the ```GPT``` (GUID Partition Table) to store boot information.
- Network boot: This method allows a computer to boot form a network server rather than from a local hard drive. ```PXE``` (Pre-boot eXecution Environment) query a server to download boot image.
- Live boot: This method allows a user to boot a computer from removable storage such as USB drive or CD/DVD or ISO image.

## Linux Kernel

The Linux kernel is the core component of the Linux operating system. It is responsble for managing sytem resources, providing access to hardware devices and facilating communication between software components. Its task include, process management, memory management, device drivers and security features. 

The Linux kernel is also an open-source software project that is develped by a large community of develpers around the world. It is released under the GNU General Public License.

## Systemd

Systemd is a system and service manager for Linux operating systems. Its a replacement for the traditional System V init system. Systemd provides features like on-demand starting of daemons, dependency-based service control logic, sockets and D-Bus activation fro starting services and parallelisation of service startup. It is responsible for managing the entire system startup process, which includes starting and stopping system services, managing system targets, and handling system events.

## Linux boot process

- POST (Power-On Self-Test): On startup, the BIOS or UEFI firmware performs a POST to verify that the hardware components are functioning correctly.
- Bootloader: The boot loader is responsible for loading the Linux kernel into memory. 
- Kernel initialisation: Once the kernel is loaded, it begins initialising the system by identifying hardware components, and loading necessary drivers. The kernel also creates  the initial root file system and sets up the system's memory management.
- Initramfs (initial RAM file system): In some cases, the kernel may need to load additional drivers or other files from the disk before the root file system can be mounted. The kernel may use and initramfs to provide a temporary file system that contains the necessary files.
- Init system: Once the kernel has finished initialisation, it hands control over the system's init system, which is responsible for starting system services and user applications. The most common init system on Linux sysmtem is systemd.
- Userspace: After the init system has started, the system enters user space, where the user can interact with the system through a GUI or CLI.

## Networks

| CLI commands | Description                                                                                                  |
| -------------|--------------------------------------------------------------------------------------------------------------|
| ip add       | Display ip address for network devices. eg: eth0, wlan0                                                      |
| ping         | Sends a small packet to destination ip address and waits for acknowledgemnt to check if network is up.       |
| ip route     | Displays the routing table for network devices.                                                              |
| dig          | Used to retrieve dns records like A, AAAA, TXT, CNAME, SOA ...                                               |

### Network config files

```
- /etc/hosts          : Local file for dns lookup.
- /etc/resolv.conf    : Lookup nameserver. //do not edit
- /etc/nsswitch.conf  : Change order of dns lookup.
- /etc/sysconfig/      : Unique to RedHat i.e CENT OS
```

## Storage

Partitioning schemes:
- MBR (Master Boot Record): It is a leagacy partitioning scheme which has supports for up to four primary partitions or threee primary partititions an done extended partition. MBR uses 32-bit disk sector addressing scheme, which limits the maximum disk size to 2 terabytes (TB).
- GPT (GUID Partition Table): It is a newer partitioning scheme designed to replace MBR which uses 64-bit disk addressing scheme, which allows for much larger disk sizes, up to 9.4 zettabytes (ZB). GPT also supports upto 128 partitions and has other features like disk recovery in case of disk failure.

### Linux File System Hierarchy

```
/bin   : This directory contains essential system binaries. eg: ls, mv, rm
/sbin  : This directory contains binary for system administration. eg: iptables
/usr   : This directory contains user binaries, libraries, documentations and other files that a user has installed.
/etc   : This directory contains configurarion files for the system.
/var   : This directory contains variable data files, such as logs files.
/tmp   : This is a directory for storing temporary files.
/home  : This directory conatins uer home directories.
/root  : This is the root user home directory
/dev   : This directory contains device files, which are used by the system to communicate.
/proc  : This directory contains virtual files that provide information about the system.
```

| CLI commands | Description                                                                                                  |
| -------------|--------------------------------------------------------------------------------------------------------------|
| lsblk        | Display all block devices in the linux system, like disks and partitions.                                    |
| fdisk        | A cli tool for managing partitions, creating, resizing, deleting.                                            |
| df           | Display information about the file system disk space usage.                                                  |
| mkfs         | A tool used to create new file system on disk partition or a storage device.                                 |
| mount        | Mount a block device in the linux file system. Edit /etc/fstab for mount bootup.                             |

## File System
```
- ext   : This is the most common file system in Linux distributions. It has a journaling file system and also supports large files and partitions.
- xfs   : This is a high performance file system designed for ouse on large-scale enterprises. It can handle high levels of concurrent I/O requests.
- btrfs : This is a modern copy-on-write (CoW) file system that supports features like snapshots, subvolumes, and transparent compression.
- NTFS  : This type of file system is used in Windows OS.
- FAT32 : Filesystem found in removable media like USB drives.
- HFS+  : Filesystem of macOS.
```

### LVM (Logival Volume Manager)

LVM is a tool for managing disk storage in Linux and other Unix-like operating systems. It provides a flexible way to manage disk space by allowing administrators to create and manage logival volumes (LVs) that span multiple physical disks or partitions.
Using LVM, administrators can create logical volues that can be resized or moved wihtout the need to move data or partitions. This provides a level of flexibility that is not available with traditional disk partitioning schemes.

LVMS consists of:
- Physical volumes (PVs)
- Volume groups (VGs)
- Logical volumes (LVs)

A physical volume is a disk or disk partition that is managed by LVM. A volume group consists of a collection of physical volumes that are grouped togethere into a logical storage pool. A logical volume is a protion of a volume group that acts as a virtual disk, which can be formatted with a file stystem and mounted like a regular disk.

### RAID (Redundant Array of Inexpensive Disks)

RAID is a technology used to increase the reliability, availability, and performance of computer storage systems by combining multiple physical disk drivees into a single logical unit.

RAID types:
- RAID 0: This configuration splits data across multiple disks also known as striping, which improves performance but does not provide any redundancy. 
- RAID 1: Also known as mirroring, this configuration duplicates data across multiple disks, providing redundancy but not improving performance. If one disks fails, the other can still function.
- RAID 5: This configuration stripes data across multiple disks and uses parity to provide redundancy. If one disk fails, the data can be reconstructed using the parity information stored on the remaining disks.
- RAID 6: Similar to RAID 5, but with tow sets of parity information for increased redundancy.
- RAID 10: Also known as RAID 1+0, this configuration combines mirroring and striping to provide both redundancy and performance. DAta is duplicated across multiple disks, and the duplicates are striped for faster access.

RAID can be implemented using hardware of software.

> To be continued....

