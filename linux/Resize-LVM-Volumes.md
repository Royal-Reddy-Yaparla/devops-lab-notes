# Resize LVM Volumes on Linux EC2

This doc walks through resizing LVM-based volumes, such as `/home`, on an EC2 instance by checking partition availability, resizing disk, and extending logical volumes and the file system.

---

## Step 1: Check Disk and Partition Layout

### List block storage devices:

```bash
lsblk
```
> Shows disk, partitions, and mounted volumes (e.g., `/dev/xvda4`, `/home`, `/var`, etc.)

---
## Step 2: Check Free Space in Volume Group

Check available space inside the volume group:

```bash
sudo vgdisplay RootVG
```

Look for:

```
Free  PE / Size       512 / 2.00 GiB
```

If you see:

```
Free  PE / Size       0 / 0
```

→ It means no space is available in the volume group and **physical volume needs to be resized first.**

---

## Step 3: Resize Partition (xvda4) Using growpart

If the volume group is full (`0 / 0`), expand the partition using:

```bash
sudo growpart /dev/xvda 4
```

> This expands the partition `/dev/xvda4` to use available free space in the disk.

---

## Step 4: Extend Logical Volume

Once space is available in the VG, extend the desired logical volume (e.g., `/home`):

```bash
sudo lvextend -L +2G /dev/RootVG/homeVol
```

* `+2G`: adds 2GB to the current volume size
* Use `-L 10G` to set an absolute size if needed

---

## Step 5: Resize the File System

For XFS file systems (common on RHEL/CentOS/Amazon Linux):

```bash
sudo xfs_growfs /home
```

This resizes the filesystem to match the new logical volume size **without downtime**.

---

## Final Verification

```bash
df -hT
```

Check if `/home` has increased in size.

---