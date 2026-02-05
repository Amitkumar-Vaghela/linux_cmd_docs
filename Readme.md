# Linux & DevOps Commands – Real-Time Production Documentation

This document explains **Linux commands and LVM storage management** exactly as used by **DevOps engineers in real production environments**, including **multi-disk LVM**, **mounting**, and **daily operational commands**.

---

## REAL-TIME DEVOPS SCENARIO

An application running on an **AWS EC2 instance** is generating logs and database files rapidly.  
The root disk is getting full.

**Requirement:**
- Add multiple new disks
- Combine them into one logical storage
- Mount it safely
- Use it for application data **without downtime**

---

# SECTION 1: DISK & LVM STORAGE MANAGEMENT (CORE DEVOPS SKILL)

## Step 1: Check Available Disks

```bash
lsblk
```

Real-Time Use Case:
Identify newly attached EBS volumes (/dev/xvdf, /dev/xvdg, /dev/xvdh) in EC2.

Step 2: Create Physical Volumes (PV)
```
pvcreate /dev/xvdf /dev/xvdg /dev/xvdh
```

Step 3: Verify Physical Volumes
``pvs
``
Why:
Confirm disk sizes and availability before grouping.

Step 4: Create Volume Group (VG) Using Multiple Disks
```
vgcreate tws_vg /dev/xvdf /dev/xvdg
```

What it does:
Combines multiple disks into a single storage pool.

DevOps Scenario:
Increase capacity by pooling disks instead of resizing one disk.

Step 5: Verify Volume Group
```
vgs
```
Why:
Ensure correct total size and free space.

Step 6: Create Logical Volume (LV)
```
lvcreate -L 10G -n tws_lv tws_vg
```
What it does:
Creates a usable logical disk for applications.

DevOps Scenario:
Allocate fixed storage for app logs, Docker data, or databases.

Step 7: Verify Logical Volume
```
lvs
```

Step 8: Format the Logical Volume
```
mkfs.ext4 /dev/tws_vg/tws_lv
```

Why formatting is required:
Linux cannot store data until a filesystem is created.

Step 9: Create Mount Directory
```
mkdir /mnt/tws_lv_mount
```

Step 10: Mount the Logical Volume (Dynamic Mount)
```
mount /dev/tws_vg/tws_lv /mnt/tws_lv_mount
```
Real-Time DevOps Use:
Temporary or testing environment storage.

⚠️ Note: This mount will be lost after reboot.

Step 11: Persistent Mount (Production Setup)
Get UUID (Best Practice)
```
blkid /dev/tws_vg/tws_lv
```
Edit fstab
```
vim /etc/fstab
```

Add entry:
```
UUID=<your-uuid>  /mnt/tws_lv_mount  ext4  defaults  0  0
```

Apply Configuration
```
mount -a
```

DevOps Scenario:
Ensure application storage survives server reboot.

SECTION 2: USER & GROUP MANAGEMENT
