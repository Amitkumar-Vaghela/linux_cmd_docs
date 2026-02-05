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
sudo
```
sudo systemctl restart nginx
```

Scenario: Secure admin access in production.

useradd / userdel
```
useradd devuser
userdel devuser
```

Scenario: Onboarding and offboarding engineers.

groupadd / gpasswd
```
groupadd devops
gpasswd -a devuser devops
```
Scenario: Grant Docker, Jenkins, or sudo access.

SECTION 3: FILE PERMISSIONS
```
ls -l
ls -l
```

chmod
```
chmod 755 deploy.sh
```


Scenario: Make CI/CD scripts executable.

chown / chgrp
```
chown devuser:devops app.log
```

Scenario: Fix permission issues in production apps.

umask
```
umask
```
Scenario: Secure default file permissions.


SECTION 4: FILE TRANSFER & COMPRESSION
scp
```
scp app.conf ec2-user@server:/etc/app/
```

Scenario: Copy configs to EC2.
rsync
```
rsync -av app/ /var/www/app/
```

Scenario: Zero-downtime application sync.
zip / gzip / tar
```
tar -cvf logs.tar /var/log
gzip logs.tar
```

Scenario: Compress logs before backup.

SECTION 5: NETWORKING & TROUBLESHOOTING
ifconfig / ip a
```
ip a
```
Scenario: EC2 instance not reachable.

ping
```
ping google.com
```

traceroute / tracepath / mtr
```
mtr google.com
```


Scenario: Detect network latency or packet loss.


nslookup
```
nslookup example.com
```

Scenario: Debug DNS issues.

telnet
```
telnet localhost 8080
```

Scenario: Check if application port is open.

curl vs wget
```
curl http://localhost:8080
wget http://example.com/file.tar
```

Tool	Use
curl	API testing
wget	File download


SECTION 6: PROCESS & SYSTEM MONITORING
ps
```
ps -ef
```

top
```
top
```
kill
```
kill -9 PID
```

Scenario: Kill hung processes during outages.

df / du / free / uptime
```
df -h
free -m
```


Scenario: Check system health before deployment.
