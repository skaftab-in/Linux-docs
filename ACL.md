# 🔐 Complete Linux ACL (Access Control Lists) Guide

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [Understanding ACLs vs Traditional Permissions](#understanding-acls-vs-traditional-permissions)
3. [Prerequisites and Setup](#prerequisites-and-setup)
4. [Complete ACL Workflow](#complete-acl-workflow)
5. [Essential ACL Commands](#essential-acl-commands)
6. [Practical Examples](#practical-examples)
7. [Monitoring and Verification](#monitoring-and-verification)
8. [Best Practices](#best-practices)
9. [Troubleshooting](#troubleshooting)

---

## 🎯 Introduction

Access Control Lists (ACLs) provide a more flexible and granular way to manage file and directory permissions beyond the traditional Unix permission model (owner, group, other). ACLs allow you to grant specific permissions to individual users and groups without changing the file's ownership.

### Real-World Scenario
In this guide, we'll demonstrate ACL management using a practical scenario where:
- **System**: Ubuntu VM (root access)
- **Scenario**: Setting up fine-grained permissions for user `skaftab` on files owned by `root`
- **Goal**: Grant specific read/write permissions to individual users without changing ownership

---

## 🔄 Understanding ACLs vs Traditional Permissions

### Traditional Unix Permissions
```bash
-rw-r--r-- 1 root root 0 Aug  9 02:36 data
```
- **Owner (root)**: read, write
- **Group (root)**: read only  
- **Others**: read only

### ACL Enhanced Permissions
```bash
-rw-rw-r--+ 1 root root 0 Aug  9 02:36 data
```
- **+** symbol indicates extended ACL permissions
- Can grant specific permissions to multiple users/groups
- More granular control without ownership changes

---

## ⚙️ Prerequisites and Setup

### 1. Check ACL Support
```bash
# Verify filesystem supports ACLs
mount | grep acl
tune2fs -l /dev/sda1 | grep acl
```

### 2. Install ACL Tools
```bash
# Ubuntu/Debian
sudo apt-get install acl

# RHEL/CentOS
sudo yum install acl
```

### 3. Mount with ACL Support (if needed)
```bash
# Temporary mount with ACL
sudo mount -o remount,acl /

# Permanent - add to /etc/fstab
/dev/sda1 / ext4 defaults,acl 0 1
```

---

## 🚀 Complete ACL Workflow

### Step 1: Create Test Environment
```bash
# Create directory and file as root
mkdir /home/skaftab/imp
cd /home/skaftab/imp
touch data
```

### Step 2: Check Initial Permissions
```bash
# View traditional permissions
ls -la data
# Output: -rw-r--r-- 1 root root 0 Aug  9 02:36 data

# View ACL permissions  
getfacl data
```

**Initial ACL Output:**
```
# file: data
# owner: root
# group: root
user::rw-
group::r--
other::r--
```

### Step 3: Grant User-Specific Permissions
```bash
# Grant write permission to user skaftab
setfacl -m u:skaftab:w /home/skaftab/imp/data

# Verify changes
getfacl data
ls -ltr data
```

**ACL Output After Write Permission:**
```
# file: data
# owner: root  
# group: root
user::rw-
user:skaftab:-w-    # User skaftab has write-only
group::r--
mask::rw-           # Mask limits effective permissions
other::r--
```

**File Listing Shows ACL:**
```
-rw-rw-r--+ 1 root root 0 Aug  9 02:36 data
```
*Notice the `+` indicating extended ACLs*

### Step 4: Modify User Permissions
```bash
# Grant read+write to user skaftab
setfacl -m u:skaftab:rw /home/skaftab/imp/data

# Verify updated permissions
getfacl data
```

**Updated ACL Output:**
```
# file: data
# owner: root
# group: root  
user::rw-
user:skaftab:rw-    # Now has read+write
group::r--
mask::rw-
other::r--
```

### Step 5: Grant Group Permissions
```bash
# Grant read+write to group skaftab
setfacl -m g:skaftab:rw /home/skaftab/imp/data

# Verify group ACL addition
getfacl data
```

**ACL with Group Permissions:**
```
# file: data
# owner: root
# group: root
user::rw-
user:skaftab:rw-
group::r--
group:skaftab:rw-   # Group skaftab added
mask::rw-
other::r--
```

### Step 6: Remove Specific ACL Entries
```bash
# Remove user skaftab ACL entry
setfacl -x u:skaftab /home/skaftab/imp/data

# Verify removal
getfacl data
```

**ACL After User Removal:**
```
# file: data
# owner: root
# group: root
user::rw-           # User skaftab entry removed
group::r--
group:skaftab:rw-   # Group entry remains
mask::rw-
other::r--
```

---

## 🛠️ Essential ACL Commands

### `getfacl` - View ACL Permissions

```bash
# Basic ACL display
getfacl filename

# Show ACLs for multiple files
getfacl file1 file2 file3

# Recursive ACL display
getfacl -R /path/to/directory

# Numeric output format
getfacl -n filename

# Skip files with no ACLs
getfacl -s filename
```

**Output Format Explanation:**
```
# file: data
# owner: root          # File owner
# group: root          # File group
user::rw-             # Owner permissions
user:username:rw-     # Named user permissions
group::r--            # Owning group permissions  
group:groupname:rw-   # Named group permissions
mask::rw-             # Effective permission mask
other::r--            # Other users permissions
```

### `setfacl` - Modify ACL Permissions

#### Grant Permissions (`-m` modify)
```bash
# Grant user permissions
setfacl -m u:username:rwx filename
setfacl -m u:skaftab:r-- filename        # Read only
setfacl -m u:skaftab:rw- filename        # Read+Write
setfacl -m u:skaftab:rwx filename        # Full access

# Grant group permissions  
setfacl -m g:groupname:rwx filename
setfacl -m g:developers:rw- filename

# Grant permissions to multiple entities
setfacl -m u:user1:rw-,g:group1:r-- filename

# Recursive permission setting
setfacl -R -m u:username:rwx directory/
```

#### Remove Permissions (`-x` remove)
```bash
# Remove specific user ACL
setfacl -x u:username filename

# Remove specific group ACL
setfacl -x g:groupname filename

# Remove multiple ACL entries
setfacl -x u:user1,g:group1 filename
```

#### Remove All ACLs (`-b` remove all)
```bash
# Remove all extended ACLs (keep base ACLs)
setfacl -b filename

# Recursive removal
setfacl -R -b directory/
```

#### Set Complete ACL (`--set`)
```bash
# Replace entire ACL
setfacl --set u::rw-,g::r--,o::r--,u:skaftab:rwx filename

# Set from file
setfacl --set-file=acl_rules.txt filename
```

### ACL Permission Notation
- **r** = read (4)
- **w** = write (2)  
- **x** = execute (1)
- **-** = no permission

**Examples:**
- `rwx` = read, write, execute (7)
- `rw-` = read, write only (6)  
- `r--` = read only (4)
- `-wx` = write, execute (3)

---

## 📊 Practical Examples

### Example 1: Multi-User Project Directory
```bash
# Create project directory
mkdir /projects/webapp
cd /projects/webapp
touch app.py config.ini

# Set base permissions
chmod 750 /projects/webapp
chmod 640 *.py *.ini

# Grant team access
setfacl -m u:developer1:rwx /projects/webapp
setfacl -m u:developer2:rwx /projects/webapp  
setfacl -m u:tester1:r-x /projects/webapp

# Grant file-specific permissions
setfacl -m u:developer1:rw- app.py
setfacl -m u:developer2:rw- app.py
setfacl -m u:tester1:r-- app.py

# Verify setup
getfacl /projects/webapp
getfacl app.py
```

### Example 2: Default ACLs for Directories
```bash
# Set default ACLs for new files in directory
setfacl -d -m u:skaftab:rw- /shared/documents
setfacl -d -m g:editors:rw- /shared/documents

# Test default ACL
cd /shared/documents
touch newfile.txt

# Check inherited permissions
getfacl newfile.txt
```

### Example 3: Backup and Restore ACLs
```bash
# Backup ACLs to file
getfacl -R /important/data > /backup/data_acls.txt

# Restore ACLs from backup
cd /important/data
setfacl --restore=/backup/data_acls.txt

# Copy ACLs between files
getfacl source_file | setfacl --set-file=- target_file
```

---

## 🔍 Monitoring and Verification

### Check ACL Status
```bash
# Find files with ACLs
find /path -type f -exec ls -la {} \; | grep "+"

# List files with extended attributes
find /path -type f | xargs ls -la | grep "+"

# Count files with ACLs
find /path -type f -exec getfacl {} \; 2>/dev/null | grep -c "^# file:"
```

### Effective Permissions Analysis
```bash
# Check effective permissions for user
getfacl filename | grep "user:username"

# Test actual access (as specific user)
sudo -u username test -r filename && echo "Read: YES" || echo "Read: NO"  
sudo -u username test -w filename && echo "Write: YES" || echo "Write: NO"
sudo -u username test -x filename && echo "Execute: YES" || echo "Execute: NO"
```

### ACL vs Traditional Permissions
```bash
# Compare traditional vs ACL permissions
ls -la filename                    # Traditional view
getfacl filename                   # ACL view
stat -c "%A %a %n" filename        # Octal permissions
```

### System-Wide ACL Audit
```bash
# Find all files with ACLs
find / -type f 2>/dev/null | xargs ls -la 2>/dev/null | grep "+" | head -20

# Check ACL support on filesystems
grep acl /proc/mounts

# Verify ACL tools installation
which getfacl setfacl
getfacl --version
```

---

## ✅ Best Practices

### 1. **Planning and Documentation**
- Document ACL policies before implementation
- Maintain ACL backup files for critical systems
- Use consistent naming conventions for users/groups

### 2. **Security Guidelines**
```bash
# Always use principle of least privilege
setfacl -m u:user:r-- file        # Grant minimum needed access

# Regularly audit ACL permissions
find /sensitive -exec getfacl {} \; > /audit/acl_audit_$(date +%Y%m%d).txt

# Remove unnecessary ACLs
setfacl -b old_file               # Clean up when no longer needed
```

### 3. **Performance Considerations**
- ACLs add slight overhead to filesystem operations
- Use traditional permissions for simple scenarios
- Limit ACL complexity on frequently accessed files

### 4. **Backup and Migration**
```bash
# Include ACLs in backups
tar --acls -czf backup.tar.gz /important/data

# Use rsync with ACL preservation
rsync -av --acls source/ destination/

# cp command with ACL preservation
cp -a --preserve=all source dest
```

### 5. **Default ACL Strategy**
```bash
# Set sensible defaults for shared directories
setfacl -d -m u::rwx,g::rx,o::--- /shared/workspace
setfacl -d -m g:team:rwx /shared/workspace
```

---

## 🚨 Troubleshooting

### Common Issues and Solutions

#### 1. **ACLs Not Working**
```bash
# Check filesystem ACL support
mount | grep $(df . | tail -1 | awk '{print $1}') | grep acl

# Remount with ACL support
sudo mount -o remount,acl /

# Check if file has ACLs
ls -la filename | grep "+"
```

#### 2. **Permission Denied Despite ACL**
```bash
# Check effective permissions (mask)
getfacl filename | grep mask

# Verify mask allows intended permissions
setfacl -m m::rwx filename        # Adjust mask if needed

# Check parent directory permissions
namei -mo /full/path/to/file
```

#### 3. **ACL Inheritance Issues**
```bash
# Check default ACLs on parent directory
getfacl /parent/directory | grep default

# Set proper default ACLs
setfacl -d -m u:user:rwx /parent/directory

# Test inheritance
touch /parent/directory/testfile
getfacl /parent/directory/testfile
```

#### 4. **Backup/Restore Problems**
```bash
# Backup with proper format
getfacl -R /data > acl_backup.txt

# Check backup file format
head -20 acl_backup.txt

# Restore step by step
setfacl --restore=acl_backup.txt
```

#### 5. **Performance Issues**
```bash
# Check if too many ACL entries
getfacl /slow/file | wc -l

# Consider simplifying ACLs
setfacl -b /slow/file             # Remove and rebuild simpler ACLs
```

### Error Messages and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `Operation not supported` | Filesystem lacks ACL support | Mount with `-o acl` or check filesystem type |
| `Invalid argument` | Incorrect ACL syntax | Check user/group names and permission format |
| `No such file or directory` | Nonexistent user/group | Verify user/group exists with `id username` |
| `Permission denied` | Insufficient privileges | Run as root or file owner |
| `Disk quota exceeded` | Filesystem full | Free disk space or increase quota |

### Diagnostic Commands
```bash
# Full system ACL diagnostic
echo "=== ACL Support Check ==="
grep acl /proc/mounts
echo -e "\n=== ACL Tools ==="
which getfacl setfacl
echo -e "\n=== Sample File ACLs ==="  
find /home -maxdepth 2 -type f -exec ls -la {} \; 2>/dev/null | grep "+" | head -5
```

---

## 📝 Quick Reference

### Essential Commands Summary
```bash
# View ACLs
getfacl filename                   # Show file ACLs
getfacl -R directory              # Recursive ACL display

# Set ACLs  
setfacl -m u:user:rwx file        # Grant user permissions
setfacl -m g:group:rw- file       # Grant group permissions
setfacl -d -m u:user:rwx dir      # Set default ACL

# Remove ACLs
setfacl -x u:user file            # Remove user ACL
setfacl -b file                   # Remove all extended ACLs

# Backup/Restore
getfacl -R /data > backup.acl     # Backup ACLs
setfacl --restore=backup.acl      # Restore ACLs
```

### Permission Quick Reference
- `r--` = Read only (4)
- `rw-` = Read + Write (6)  
- `rwx` = Full access (7)
- `-wx` = Write + Execute (3)
- `--x` = Execute only (1)

---

## 🎉 Conclusion

ACLs provide powerful, granular permission control beyond traditional Unix permissions. They're essential for:
- Multi-user environments
- Complex permission requirements  
- Fine-grained access control
- Enterprise security policies

Remember to always test ACL changes in a safe environment and maintain proper backups of your ACL configurations.

**Key Takeaways:**
1. ACLs extend traditional permissions without changing ownership
2. The `+` symbol in `ls -la` indicates extended ACLs
3. The mask limits effective permissions for named users/groups
4. Default ACLs apply to new files in directories
5. Regular auditing and cleanup prevent ACL sprawl

For more advanced ACL management, consider integrating with LDAP, Active Directory, or other enterprise identity management systems.
