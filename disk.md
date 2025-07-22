# 💾 Disk Management in Linux

A complete guide to disk partitioning, mounting, and filesystem management with hands-on examples from CentOS Stream.

## 🎯 Complete Disk Management Workflow

Based on the practical example of adding a new 3GB disk (`/dev/nvme0n2`) to a CentOS Stream system:

### **Step-by-Step Process:**
1. **Identify Available Disks** → `lsblk`, `sudo fdisk -l`
2. **Create Partition** → `sudo fdisk /dev/nvme0n2`
3. **Format Filesystem** → `mkfs.xfs /dev/nvme0n2p1`
4. **Create Mount Point** → `mkdir /root/new-storage`
5. **Mount Filesystem** → `mount /dev/nvme0n2p1 /root/new-storage`
6. **Make Permanent** → Edit `/etc/fstab`
7. **Reload SystemD** → `systemctl daemon-reload`
8. **Verify** → `df -h`, `mount | grep nvme0n2p1`

---

## 📁 File System & Storage Commands

### `df` - Display Filesystem Disk Space Usage
Shows disk space usage for mounted filesystems.







```bash
# Basic usage
df
```



**Practical Example -**
```bash
[skaftab@localhost ~]$ df
Filesystem          1K-blocks    Used Available Use% Mounted on
/dev/mapper/cs-root  45793280 5457748  40335532  12% /
devtmpfs                 4096       0      4096   0% /dev
tmpfs                 1853356      84   1853272   1% /dev/shm
tmpfs                  741344    9908    731436   2% /run
tmpfs                    1024       0      1024   0% /run/credentials/systemd-journald.service
/dev/nvme0n1p2         983040  341676    641364  35% /boot
/dev/mapper/cs-home  22323200  467664  21855536   3% /home
tmpfs                  370668     148    370520   1% /run/user/1000
/dev/sr0              7936518 7936518         0 100% /run/media/skaftab/CentOS-Stream-10-BaseOS-x86_64

```

```bash
# Human-readable format (recommended)
df -h
```

**Practical Example -**
```bash
[skaftab@localhost ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   44G  5.3G   39G  12% /
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                1.8G   84K  1.8G   1% /dev/shm
tmpfs                724M  9.7M  715M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/nvme0n1p2       960M  334M  627M  35% /boot
/dev/mapper/cs-home   22G  457M   21G   3% /home
tmpfs                362M  148K  362M   1% /run/user/1000
/dev/sr0             7.6G  7.6G     0 100% /run/media/skaftab/CentOS-Stream-10-BaseOS-x86_64

```



---

### `fdisk` - Manipulate Disk Partition Tables
Powerful tool for creating, deleting, and managing partitions.

```bash
# List all partitions (requires sudo)
sudo fdisk -l
```
```bash
[skaftab@localhost ~]$ sudo fdisk -l
[sudo] password for skaftab:
Disk /dev/nvme0n1: 70 GiB, 75161927680 bytes, 146800640 sectors
Disk model: VMware Virtual NVMe Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 45C6AD82-34AD-4383-BB75-715EDF5108B1

Device           Start       End   Sectors Size Type
/dev/nvme0n1p1    2048      4095      2048   1M BIOS boot
/dev/nvme0n1p2    4096   2101247   2097152   1G Linux extended boot
/dev/nvme0n1p3 2101248 146798591 144697344  69G Linux LVM


Disk /dev/nvme0n2: 3 GiB, 3221225472 bytes, 6291456 sectors
Disk model: VMware Virtual NVMe Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/nvme0n3: 2 GiB, 2147483648 bytes, 4194304 sectors
Disk model: VMware Virtual NVMe Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/cs-root: 43.73 GiB, 46959427584 bytes, 91717632 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/cs-swap: 3.91 GiB, 4198498304 bytes, 8200192 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/cs-home: 21.35 GiB, 22926065664 bytes, 44777472 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes



# Interactive partition editor for a specific device
sudo fdisk /dev/nvme0n2

```

**Practical Example - Creating a New Partition:**
```bash
# Start fdisk on the new disk
sudo fdisk /dev/nvme0n2

# Inside fdisk:
Command (m for help): n          # Create new partition
Partition type: p                # Primary partition
Partition number: 1              # First partition
First sector: <Enter>            # Use default (2048)
Last sector: <Enter>             # Use entire disk
Command (m for help): w          # Write changes to disk
```
```
[root@localhost ~]# fdisk /dev/nvme0n2

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS (MBR) disklabel with disk identifier 0x07700ffc.

Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition
   e   resize a partition

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty MBR (DOS) partition table
   s   create a new empty Sun partition table


Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-6291455, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-6291455, default 6291455):

Created a new partition 1 of type 'Linux' and of size 3 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

```

**Common fdisk Interactive Commands:**
- `m` - Show help menu
- `n` - Add new partition
- `p` - Print partition table
- `w` - Write changes to disk
- `q` - Quit without saving

---

### `lsblk` - List Block Devices
Tree view of all block devices and their mount points.

```bash
lsblk
```

**Real Output from Terminal Session:**
```
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
nvme0n1     259:0    0   70G  0 disk                    ← Main system disk
├─nvme0n1p1 259:1    0    1M  0 part                    ← BIOS boot
├─nvme0n1p2 259:2    0    1G  0 part /boot              ← Boot partition
└─nvme0n1p3 259:3    0   69G  0 part                    ← LVM physical volume
  ├─cs-root 253:0    0 43.7G  0 lvm  /                  ← Root filesystem
  ├─cs-swap 253:1    0  3.9G  0 lvm  [SWAP]            ← Swap space
  └─cs-home 253:2    0 21.4G  0 lvm  /home             ← Home directory
nvme0n2     259:4    0    3G  0 disk                    ← New disk added
└─nvme0n2p1 259:5    0    3G  0 part /root/new-storage  ← New partition mounted
nvme0n3     259:6    0    2G  0 disk                    ← Unused disk
```

---

## 🔧 File System Management

### `mkfs` - Make Filesystem
Create a filesystem on a partition.

```bash
# Create XFS filesystem (used in our example)
mkfs.xfs /dev/nvme0n2p1
mkfs.ext4 /dev/nvme0n2p1
```
# Create ext4 filesystem



# List available filesystem types
mkfs<TAB><TAB>


**Real Terminal Output:**
```bash
[root@localhost ~]# mkfs.xfs /dev/nvme0n2p1
meta-data=/dev/nvme0n2p1         isize=512    agcount=4, agsize=196544 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
data     =                       bsize=4096   blocks=786176, imaxpct=25
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

---

### `mount` - Mount Filesystems
Attach filesystems to the directory tree.

```bash
# Mount a device to a directory
mount /dev/nvme0n2p1 /root/new-storage

# Mount with specific filesystem type
mount -t xfs /dev/nvme0n2p1 /root/new-storage

# Show all mounted filesystems
mount

# Show specific mount (verify it worked)
mount | grep nvme0n2p1
```

**Verification Output:**
```bash
/dev/nvme0n2p1 on /root/new-storage type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
```

---



---

## 📝 Configuration Files

### `/etc/fstab` - Filesystem Table
Permanent mount configuration file for persistent mounts across reboots.

```bash
# Edit fstab (use your preferred editor)
vi /etc/fstab

# View current fstab
cat /etc/fstab
```

**Real fstab Entry from Terminal Session:**
```bash
# After editing /etc/fstab, the new entry was added:
/dev/nvme0n2p1  /root/new-storage    xfs    defaults    0    0
```

**Complete fstab Example:**
```
#
# /etc/fstab
# Created by anaconda on Tue Jul  1 20:11:41 2025
#
UUID=8efdfab9-1f10-4ed7-881c-e214bbd08131 /                xfs     defaults        0 0
UUID=2852b8c1-8678-49cd-a6ee-b917e2c0b7fb /boot            xfs     defaults        0 0
UUID=1e7b151a-ed0f-41d8-a6d9-ebfba9d5ef13 /home            xfs     defaults        0 0
UUID=2f4a0dbb-34be-47d2-b0f3-2b38fb081964 none             swap    defaults        0 0
/dev/nvme0n2p1                            /root/new-storage xfs     defaults        0 0
```

**fstab Entry Format:**
```
# Device/UUID    Mount Point         FS Type    Options    Dump    Pass
/dev/nvme0n2p1   /root/new-storage   xfs        defaults   0       0
```

**Field Explanations:**
- **Device**: The device or UUID to mount
- **Mount Point**: Where to mount in the filesystem
- **FS Type**: Filesystem type (xfs, ext4, etc.)
- **Options**: Mount options (defaults is most common)
- **Dump**: Backup utility flag (0 = no backup)
- **Pass**: fsck order (0 = no check, 1 = check first, 2 = check after root)

```bash
[root@localhost ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   44G  5.3G   39G  12% /
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                1.8G     0  1.8G   0% /dev/shm
tmpfs                724M  9.6M  715M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/nvme0n2p1       3.0G   90M  2.9G   3% /root/new-storage
/dev/nvme0n1p2       960M  334M  627M  35% /boot
/dev/mapper/cs-home   22G  457M   21G   3% /home
tmpfs                362M   72K  362M   1% /run/user/42
tmpfs                362M   56K  362M   1% /run/user/1000
[root@localhost ~]#

```
---









*Happy Linux Disk Management! 💾*

> **Note**: This guide is based on a real CentOS Stream 10 system with NVMe disks and XFS filesystems. Always practice these operations in a safe environment before applying to production systems.
