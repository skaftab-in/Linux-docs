# 📦 Complete Linux LVM (Logical Volume Manager) Guide

## 🎯 Introduction

Logical Volume Manager (LVM) is a powerful storage management system that provides flexibility in managing disk space. Unlike traditional partitioning, LVM allows you to resize, move, and manage storage dynamically without unmounting filesystems.

**Practical Scenario**: Setting up LVM on a CentOS Stream system with a new 5GB disk (`/dev/nvme0n2`) to create flexible storage that can be easily managed and expanded.

## 🏗️ LVM Architecture Overview

```
Physical Disk → Physical Volume (PV) → Volume Group (VG) → Logical Volume (LV) → Filesystem
```

- **Physical Volume (PV)**: Raw disk or partition prepared for LVM
- **Volume Group (VG)**: Pool of storage created from one or more PVs
- **Logical Volume (LV)**: Virtual partition created from VG space
- **Filesystem**: Formatted LV mounted to a directory

## 📋 Complete Workflow Overview

1. **System Assessment** - Check current storage layout
2. **Disk Preparation** - Partition the new disk for LVM
3. **Physical Volume Creation** - Initialize disk for LVM use
4. **Volume Group Creation** - Create storage pool
5. **Logical Volume Creation** - Create virtual partitions
6. **Filesystem Creation** - Format the logical volume
7. **Mounting & Configuration** - Mount and make persistent
8. **Verification** - Confirm setup works after reboot

---

## 🔍 Step 1: System Assessment

### Check Current Storage Layout

```bash
# View block devices hierarchy
lsblk
```

**Expected Output:**
```
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0          11:0    1  7.6G  0 rom
nvme0n1     259:0    0   70G  0 disk
├─nvme0n1p1 259:1    0    1M  0 part
├─nvme0n1p2 259:2    0    1G  0 part /boot
└─nvme0n1p3 259:3    0   69G  0 part
  ├─cs-root 253:0    0 43.7G  0 lvm  /
  ├─cs-swap 253:1    0  3.9G  0 lvm  [SWAP]
  └─cs-home 253:2    0 21.4G  0 lvm  /home
nvme0n2     259:4    0    5G  0 disk    ← Our target disk
nvme0n3     259:5    0    2G  0 disk
```

### Check Current Disk Usage

```bash
# View filesystem usage
df -h
```

**Expected Output:**
```
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   44G  4.8G   39G  11% /
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                1.8G     0  1.8G   0% /dev/shm
tmpfs                724M   12M  713M   2% /run
/dev/nvme0n1p2       960M  334M  627M  35% /boot
/dev/mapper/cs-home   22G  457M   21G   3% /home
```

### List All Available Disks

```bash
# Detailed disk information
fdisk -l
```

**Key Information from Output:**
- `nvme0n2`: 5 GiB unpartitioned disk - **Perfect for LVM**
- `nvme0n3`: 2 GiB unpartitioned disk - **Available for future use**

---

## 🔧 Step 2: Disk Preparation

### Create LVM Partition on Target Disk

```bash
# Start fdisk on the target disk
fdisk /dev/nvme0n2
```

**Interactive fdisk Session:**

```
Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS (MBR) disklabel with disk identifier 0x89756112.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-10485759, default 2048): [ENTER]
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759): [ENTER]

Created a new partition 1 of type 'Linux' and of size 5 GiB.

Command (m for help): t
Selected partition 1
Hex code or alias (type L to list all): 8e
Changed type of partition 'Linux' to 'Linux LVM'.

Command (m for help): p
Disk /dev/nvme0n2: 5 GiB, 5368709120 bytes, 10485760 sectors
Device         Boot Start      End  Sectors Size Id Type
/dev/nvme0n2p1       2048 10485759 10483712   5G 8e Linux LVM

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

### 🔑 Key fdisk Commands Explained:

- **`n`** - Create new partition
- **`p`** - Primary partition (vs extended)
- **`t`** - Change partition type
- **`8e`** - Linux LVM partition type code
- **`w`** - Write changes to disk

### Verify Partition Creation

```bash
lsblk
```

**After Partitioning:**
```
nvme0n2     259:4    0    5G  0 disk
└─nvme0n2p1 259:7    0    5G  0 part    ← New LVM partition created
```

---

## 🏗️ Step 3: Physical Volume (PV) Creation

### Create Physical Volume

```bash
# Initialize partition for LVM use
pvcreate /dev/nvme0n2p1
```

**Output:**
```
Physical volume "/dev/nvme0n2p1" successfully created.
```

### Verify Physical Volume

```bash
# List all physical volumes
pvs
```

**Output:**
```
PV             VG Fmt  Attr PSize   PFree
/dev/nvme0n1p3 cs lvm2 a--  <69.00g     0      ← Existing system PV
/dev/nvme0n2p1    lvm2 ---   <5.00g <5.00g    ← Our new PV (not in VG yet)
```

### 📊 PV Status Explanation:

- **VG**: Volume Group name (empty means not assigned)
- **Fmt**: LVM format version
- **Attr**: Attributes (a=allocatable, -=not in VG)
- **PSize**: Physical size
- **PFree**: Free space available

---

## 🗂️ Step 4: Volume Group (VG) Creation

### Create Volume Group

```bash
# Create VG named "new_VG" using our PV
vgcreate new_VG /dev/nvme0n2p1
```

**Output:**
```
Volume group "new_VG" successfully created
```

### Verify Volume Group

```bash
# List all volume groups
vgs
```

**Output:**
```
VG     #PV #LV #SN Attr   VSize   VFree
cs       1   3   0 wz--n- <69.00g      0    ← System VG (full)
new_VG   1   0   0 wz--n-  <5.00g <5.00g   ← Our new VG (empty)
```

### 📈 VG Status Explanation:

- **#PV**: Number of Physical Volumes
- **#LV**: Number of Logical Volumes
- **#SN**: Number of Snapshots
- **Attr**: Attributes (w=writeable, z=resizable, n=normal)
- **VSize**: Total size
- **VFree**: Free space available

---

## 💾 Step 5: Logical Volume (LV) Creation

### Create Logical Volume

```bash
# Create LV named "new_LV" with 5000MB from "new_VG"
lvcreate -n new_LV --size 5000 new_VG
```

**Output:**
```
Logical volume "new_LV" created.
```

### Verify Logical Volume Creation

```bash
# Check VG status after LV creation
vgs
```

**Output:**
```
VG     #PV #LV #SN Attr   VSize   VFree
cs       1   3   0 wz--n- <69.00g      0
new_VG   1   1   0 wz--n-  <5.00g 116.00m   ← Now has 1 LV, 116MB free
```

### 🎯 LV Size Calculation:

- **Requested**: 5000MB (5000 × 1MB)
- **Available**: ~5.00GB (5120MB)
- **Remaining**: 116MB (5120 - 5000 - metadata overhead)

---

## 🗃️ Step 6: Filesystem Creation

### Format the Logical Volume

```bash
# Create XFS filesystem on the LV
mkfs.xfs /dev/new_VG/new_LV
```

**Output:**
```
meta-data=/dev/new_VG/new_LV     isize=512    agcount=4, agsize=320000 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
data     =                       bsize=4096   blocks=1280000, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

### 📁 Filesystem Details:

- **Block size**: 4096 bytes (4KB)
- **Total blocks**: 1,280,000 (≈ 4.88GB usable)
- **Allocation groups**: 4 (for parallel I/O)
- **Features**: CRC, finobt, sparse, rmapbt, reflink enabled

---

## 🔗 Step 7: Mounting & Configuration

### Create Mount Point

```bash
# Create directory for mounting
mkdir /new_folder
```

### Mount the Logical Volume

```bash
# Mount the LV to the directory
mount /dev/new_VG/new_LV /new_folder
```

### Verify Mount

```bash
df -h
```

**Output:**
```
Filesystem                 Size  Used Avail Use% Mounted on
/dev/mapper/cs-root         44G  4.8G   39G  11% /
/dev/nvme0n1p2             960M  334M  627M  35% /boot
/dev/mapper/cs-home         22G  457M   21G   3% /home
/dev/mapper/new_VG-new_LV  4.9G  127M  4.7G   3% /new_folder  ← Our new LV mounted!
```

### Get UUID for Persistent Mounting

```bash
# Get filesystem UUIDs
blkid
```

**Key Output:**
```
/dev/mapper/new_VG-new_LV: UUID="5b65efd5-c0e2-438c-9955-2c555081ec47" BLOCK_SIZE="512" TYPE="xfs"
```

### Configure Persistent Mounting

```bash
# Edit fstab for automatic mounting
vi /etc/fstab
```

**Add this line to /etc/fstab:**
```bash
# My new LVM filesystem
UUID="5b65efd5-c0e2-438c-9955-2c555081ec47"     /new_folder     xfs     defaults        0 0

# Alternative using device path (less preferred):
# /dev/mapper/new_VG-new_LV     /new_folder     xfs     defaults        0 0
```

### 🔄 fstab Entry Explanation:

- **UUID**: Unique identifier (preferred over device names)
- **Mount point**: `/new_folder`
- **Filesystem type**: `xfs`
- **Mount options**: `defaults` (rw,suid,dev,exec,auto,nouser,async)
- **Dump**: `0` (no backup)
- **Pass**: `0` (no fsck check)

---

## ✅ Step 8: Verification & Testing

### Test Reboot Persistence

```bash
# Reboot system to test persistence
init 6
```

### After Reboot - Verify Auto-Mount

```bash
# Check if LV mounted automatically
df -h
```

**Success! Output shows:**
```
Filesystem                 Size  Used Avail Use% Mounted on
/dev/mapper/cs-root         44G  4.8G   39G  11% /
/dev/nvme0n1p2             960M  334M  627M  35% /boot
/dev/mapper/cs-home         22G  457M   21G   3% /home
/dev/mapper/new_VG-new_LV  4.9G  127M  4.7G   3% /new_folder  ← ✅ Mounted automatically!
```

---

## 🛠️ Essential LVM Management Commands

### Physical Volume Commands

```bash
# Create PV
pvcreate /dev/device

# Display PV information
pvs                    # Summary view
pvdisplay             # Detailed view

# Remove PV (must be unused)
pvremove /dev/device
```

### Volume Group Commands

```bash
# Create VG
vgcreate vg_name /dev/pv1 /dev/pv2

# Display VG information
vgs                    # Summary view
vgdisplay             # Detailed view

# Extend VG (add PV)
vgextend vg_name /dev/new_pv

# Remove VG
vgremove vg_name
```

### Logical Volume Commands

```bash
# Create LV
lvcreate -n lv_name -L size vg_name
lvcreate -n lv_name -l 100%FREE vg_name    # Use all free space

# Display LV information
lvs                    # Summary view
lvdisplay             # Detailed view

# Extend LV
lvextend -L +1G /dev/vg/lv                 # Add 1GB
lvextend -l +100%FREE /dev/vg/lv           # Use all free space

# Remove LV (must be unmounted)
lvremove /dev/vg/lv
```

---

## 🔧 Advanced LVM Operations - Extending Storage

### Real-World Extension Scenario

**Situation**: Your `/new_folder` storage is getting full and you have a new 2GB disk (`/dev/nvme0n3`) to add.

**Current State Check:**
```bash
# Check current LVM status
pvs
vgs  
lvs
df -h
```

**Output shows:**
```bash
# Physical Volumes
PV             VG     Fmt  Attr PSize   PFree
/dev/nvme0n1p3 cs     lvm2 a--  <69.00g      0
/dev/nvme0n2p1 new_VG lvm2 a--   <5.00g 116.00m

# Volume Groups  
VG     #PV #LV #SN Attr   VSize   VFree
cs       1   3   0 wz--n- <69.00g      0
new_VG   1   1   0 wz--n-  <5.00g 116.00m  ← Only 116MB free!

# Logical Volumes
LV     VG     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
new_LV new_VG -wi-ao----  4.88g

# Filesystem Usage
/dev/mapper/new_VG-new_LV  4.9G  127M  4.7G   3% /new_folder
```

### Step 1: Prepare the New Disk

```bash
# Partition the new 2GB disk
fdisk /dev/nvme0n3
```

**Interactive fdisk session:**
```bash
Welcome to fdisk (util-linux 2.40.2).
Device does not contain a recognized partition table.
Created a new DOS (MBR) disklabel with disk identifier 0xb4dad457.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): [ENTER]
First sector (2048-4194303, default 2048): [ENTER] 
Last sector, +/-sectors or +/-size{K,M,G,T,P}: [ENTER]

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): t
Selected partition 1
Hex code or alias (type L to list all): 8e
Changed type of partition 'Linux' to 'Linux LVM'.

Command (m for help): p
Disk /dev/nvme0n3: 2 GiB, 2147483648 bytes, 4194304 sectors
Device         Boot Start     End Sectors Size Id Type
/dev/nvme0n3p1       2048 4194303 4192256   2G 8e Linux LVM

Command (m for help): w
The partition table has been altered.
```

**Verify new partition:**
```bash
lsblk
```
```bash
nvme0n3           259:6    0    2G  0 disk
└─nvme0n3p1       259:7    0    2G  0 part  ← New partition ready
```

### Step 2: Create Physical Volume

```bash
# Create PV from the new partition
pvcreate /dev/nvme0n3p1
```

**Output:**
```bash
Physical volume "/dev/nvme0n3p1" successfully created.
```

**Verify PV creation:**
```bash
pvs
```
```bash
PV             VG     Fmt  Attr PSize   PFree  
/dev/nvme0n1p3 cs     lvm2 a--  <69.00g      0
/dev/nvme0n2p1 new_VG lvm2 a--   <5.00g 116.00m
/dev/nvme0n3p1        lvm2 ---   <2.00g  <2.00g  ← New PV (not assigned to VG yet)
```

### Step 3: Extend Volume Group

```bash
# Add new PV to existing VG
vgextend new_VG /dev/nvme0n3p1
```

**Output:**
```bash
Volume group "new_VG" successfully extended
```

**Verify VG extension:**
```bash
vgs
```
```bash
VG     #PV #LV #SN Attr   VSize   VFree
cs       1   3   0 wz--n- <69.00g     0
new_VG   2   1   0 wz--n-   6.99g <2.11g  ← Now 2 PVs, ~2GB more space!
```

### Step 4: Extend Logical Volume

**Check current filesystem size:**
```bash
df -h | grep new_folder
```
```bash
/dev/mapper/new_VG-new_LV  4.9G  127M  4.7G   3% /new_folder  ← Before extension
```

**Extend LV to use all available space:**
```bash
# Use all remaining free space in VG
lvextend -l +100%FREE /dev/new_VG/new_LV
```

**Output:**
```bash
Size of logical volume new_VG/new_LV changed from 4.88 GiB (1250 extents) to 6.99 GiB (1790 extents).
Logical volume new_VG/new_LV successfully resized.
```

### Step 5: Extend Filesystem

**⚠️ Critical Step**: The LV is larger, but the filesystem doesn't know yet!

**For XFS filesystems:**
```bash
# Grow XFS filesystem to fill the extended LV
xfs_growfs /dev/mapper/new_VG-new_LV
```

**Output:**
```bash
meta-data=/dev/mapper/new_VG-new_LV isize=512    agcount=4, agsize=320000 blks
         =                       sectsz=512   attr=2, projid32bit=1
data     =                       bsize=4096   blocks=1280000, imaxpct=25
         =                       sunit=0      swidth=0 blks
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 1280000 to 1832960  ← Filesystem grew!
```

### Step 6: Verify Extension Success

```bash
df -h | grep new_folder
```

**After extension:**
```bash
/dev/mapper/new_VG-new_LV  7.0G  169M  6.8G   3% /new_folder  ← Extended from 4.9G to 7.0G!
```

### Step 7: Test Persistence After Reboot

```bash
# Reboot to ensure everything works
init 6
```

**After reboot verification:**
```bash
df -h | grep new_folder
```
```bash
/dev/mapper/new_VG-new_LV  7.0G  169M  6.8G   3% /new_folder  ← ✅ Extension survived reboot!
```

## 📊 Extension Summary

**Before Extension:**
- Volume Group: 5.00G (116MB free)
- Logical Volume: 4.88G  
- Filesystem: 4.9G usable

**After Extension:**
- Volume Group: 6.99G (added 2GB disk)
- Logical Volume: 6.99G (used all available space)
- Filesystem: 7.0G usable (43% increase!)

## 🔧 Alternative Extension Methods

### Method 1: Extend by Specific Size
```bash
# Add exactly 1GB to LV
lvextend -L +1G /dev/new_VG/new_LV

# Then grow filesystem
xfs_growfs /dev/mapper/new_VG-new_LV
```

### Method 2: Extend to Specific Total Size  
```bash
# Make LV exactly 8GB total
lvextend -L 8G /dev/new_VG/new_LV

# Then grow filesystem
xfs_growfs /dev/mapper/new_VG-new_LV
```

### Method 3: Extend by Percentage
```bash
# Use 50% of remaining free space
lvextend -l +50%FREE /dev/new_VG/new_LV

# Then grow filesystem  
xfs_growfs /dev/mapper/new_VG-new_LV
```

## ⚡ Filesystem-Specific Growth Commands

### For XFS (Red Hat/CentOS default):
```bash
# Grow XFS filesystem (must be mounted)
xfs_growfs /mount/point
# OR
xfs_growfs /dev/mapper/vg-lv
```

### For ext4 (Ubuntu/Debian common):
```bash
# Grow ext4 filesystem (can be mounted or unmounted)
resize2fs /dev/mapper/vg-lv
```

### For ext3:
```bash
# Same as ext4
resize2fs /dev/mapper/vg-lv
```

### LVM Snapshots

```bash
# Create snapshot (10% of original size)
lvcreate -L 500M -s -n new_LV_snapshot /dev/new_VG/new_LV

# Mount snapshot
mkdir /mnt/snapshot
mount /dev/new_VG/new_LV_snapshot /mnt/snapshot

# Remove snapshot when done
umount /mnt/snapshot
lvremove /dev/new_VG/new_LV_snapshot
```

---

## ⚠️ Troubleshooting Common Issues

### Problem: "Cannot use device: device is partitioned"

**Error when running:** `pvcreate /dev/nvme0n2`
```
Cannot use /dev/nvme0n2: device is partitioned
```

**Solution:** Use the partition, not the whole disk:
```bash
pvcreate /dev/nvme0n2p1  # Use partition, not whole disk
```

### Problem: Logical Volume Not Mounting After Reboot

**Symptoms:** LV exists but not mounted

**Check fstab syntax:**
```bash
cat /etc/fstab
# Ensure UUID is correct and no typos
```

**Verify UUID:**
```bash
blkid | grep new_VG-new_LV
```

**Test mount manually:**
```bash
mount -a  # Mount all fstab entries
```

### Problem: No Space Left in Volume Group

**Error when creating LV:**
```
Insufficient free space: X extents needed, but only Y available
```

**Check available space:**
```bash
vgs  # Check VFree column
```

**Solutions:**
1. **Extend VG:** Add more PVs
2. **Reduce LV size:** Use smaller size
3. **Remove unused LVs:** Free up space

### Problem: Cannot Remove Volume Group

**Error:**
```
Volume group "new_VG" in use
```

**Solution sequence:**
```bash
# 1. Unmount all LVs in the VG
umount /new_folder

# 2. Remove all LVs in the VG
lvremove /dev/new_VG/new_LV

# 3. Now remove the VG
vgremove new_VG

# 4. Finally remove PVs
pvremove /dev/nvme0n2p1
```

---

## 🎯 Best Practices

### 🔹 Planning & Design

- **Plan capacity**: Leave 10-20% free space in VGs for snapshots and growth
- **Use descriptive names**: `data_vg`, `backup_lv` instead of generic names
- **Document layout**: Keep records of your LVM structure

### 🔹 Security & Backup

- **Regular backups**: LVM doesn't replace backups
- **Test snapshots**: Practice snapshot creation and restoration
- **Monitor space**: Set up alerts for VG space usage

### 🔹 Performance Optimization

- **Align partitions**: Use proper sector alignment (fdisk does this automatically)
- **Choose right filesystem**: XFS for large files, ext4 for general use
- **Stripe across PVs**: For better performance with multiple disks

### 🔹 Maintenance

- **Regular monitoring**: Check `pvs`, `vgs`, `lvs` output regularly
- **Update documentation**: Keep fstab and mount points documented
- **Test recovery**: Practice LVM recovery procedures

---

## 📚 Quick Reference

### 🔍 Information Commands
```bash
lsblk                    # Block device tree
pvs / vgs / lvs          # LVM summary
pvdisplay / vgdisplay / lvdisplay  # Detailed info
df -h                    # Filesystem usage
blkid                    # UUID information
```

### 🏗️ Creation Commands
```bash
pvcreate /dev/device     # Create physical volume
vgcreate vg_name /dev/pv # Create volume group
lvcreate -n lv -L size vg # Create logical volume
mkfs.xfs /dev/vg/lv      # Create filesystem
```

### 📁 Mount Commands
```bash
mkdir /mount_point       # Create mount point
mount /dev/vg/lv /mount_point  # Mount filesystem
echo "UUID=... /mount_point xfs defaults 0 0" >> /etc/fstab  # Persistent mount
```

---

## 🎉 Conclusion

You have successfully created a complete LVM setup! Your system now has:

- ✅ **Physical Volume**: `/dev/nvme0n2p1` (5GB)
- ✅ **Volume Group**: `new_VG` (flexible storage pool)
- ✅ **Logical Volume**: `new_LV` (virtual partition)
- ✅ **XFS Filesystem**: Formatted and ready for use
- ✅ **Persistent Mount**: Automatically mounts on boot at `/new_folder`

This LVM setup provides the foundation for flexible storage management, easy expansion, and advanced features like snapshots. The system is now ready for production use and can be easily expanded as storage needs grow.

**Remember:** Always test LVM operations in a non-production environment first, and maintain regular backups regardless of your storage configuration! 🚀
