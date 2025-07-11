
# EBS Volume Mounting and Persistence on EC2

This doc  explains how to attach an EBS volume to an EC2 instance, mount it, and make the mount persistent across reboots.

---

## Step-by-Step Process

### 1. Attach the Volume to EC2

- In AWS Console, create a new EBS volume.
- Ensure it's in the same **Availability Zone** as your EC2 instance.
- Attach it to the instance (e.g., as `/dev/xvdf`).

### 2. Check Volume Attachment

SSH into your EC2 instance and verify the attached volume:

```bash
lsblk
````

You should see a new block device like `/dev/xvdf`.

---

### 3. Create a Filesystem

Format the new volume with a filesystem (XFS in this case):

```bash
sudo mkfs.xfs /dev/xvdf
```

---

### 4. Create a Mount Point

Create a directory to mount the volume:

```bash
sudo mkdir /data
```

---

### 5. Mount the Volume (Temporary Mount)

Mount the volume to the directory:

```bash
sudo mount /dev/xvdf /data
```

You can now use `/data` as storage backed by the EBS volume.

---

### 6. Create and Verify a File

```bash
echo "test data from EBS volume" > /data/file.txt
cat /data/file.txt
```

You should see the content printed.

---

### 7. Reboot Behavior

If you reboot the instance now:

* The **volume remains attached**
* But `/data` will appear **empty** because the mount is **temporary**
* You’ll need to remount the volume manually:

```bash
sudo mount /dev/xvdf /data
```

---

### 8. Make the Mount Persistent

#### Get the UUID of the volume:

```bash
sudo blkid /dev/xvdf
```

Example output:

```
/dev/xvdf: UUID="54cd7e6b-3cf3-443a-876e-1a17362295ec" TYPE="xfs"
```

#### Edit `/etc/fstab`:

```bash
sudo vi /etc/fstab
```

Add this line at the bottom:

```
UUID=54cd7e6b-3cf3-443a-876e-1a17362295ec /data xfs defaults,nofail 0 2
```

 This ensures the volume will automatically mount to `/data` on every reboot.
