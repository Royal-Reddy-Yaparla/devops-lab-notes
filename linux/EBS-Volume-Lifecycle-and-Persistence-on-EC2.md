# EBS Volume Lifecycle and Persistence on EC2
---
scenario: 
- Create a new volume for private machine, attach it and create files on volume 2 
- Create a snapshot of volume 2 with files, create a volume from that snapshot, attach it as third volume to Private VM, volume should be visible even after server reboot.


This doc walks through attaching an EBS volume to a private EC2 instance, creating and restoring a snapshot, and configuring the restored volume to persist across reboots.

---

## Step 1: Launch EC2

* Launch a new EC2 instance in your desired availability zone.

---

## Step 2: Create and Attach Volume-1

* Create a new EBS volume (1 GB) in the same availability zone as the EC2 instance.
* Attach the volume to the EC2 instance (e.g., as `/dev/xvdbb`).

---

## Step 3: Format and Mount Volume-1

SSH into your EC2 instance:

```bash
lsblk
```

You should see the new volume (e.g., `/dev/xvdbb`).

Create a filesystem:

```bash
sudo mkfs.xfs /dev/xvdbb
```

Create a mount point:

```bash
sudo mkdir /volume1
```

Mount the volume:

```bash
sudo mount /dev/xvdbb /volume1
```

---

## Step 4: Create a File on Volume-1

```bash
cd /volume1
echo "data from volume-1" > text.txt
```

---

## Step 5: Create a Snapshot of Volume-1

* In AWS Console, go to EC2 > Volumes.
* Select `volume-1` and choose **Create Snapshot**.
* Provide a snapshot name and create it.

---

## Step 6: Create Volume-2 from Snapshot

* Go to EC2 > Snapshots.
* Select the snapshot created above.
* Choose **Create Volume** from Snapshot.
* Make sure it's in the same availability zone as the EC2 instance.
* Name it `volume-2` and attach it to the EC2 instance using a different device name (e.g., `/dev/xvdbc`).

---

## Step 7: Mount Volume-2

Check if the volume is attached:

```bash
lsblk
```

Create a new mount point:

```bash
sudo mkdir /volume2
```

Mount the new volume:

```bash
sudo mount /dev/xvdbc /volume2
```

Verify the file is restored from snapshot:

```bash
ls /volume2
cat /volume2/text.txt
```

You should see the content: `data from volume-1`

---

## Step 8: Persist Volume After Reboot

Get the UUID of the volume:

```bash
sudo blkid /dev/xvdbc
```

Example output:

```
/dev/xvdbc: UUID="54cd7e6b-3cf3-443a-876e-1a17362295ec" TYPE="xfs"
```

Edit `/etc/fstab`:

```bash
sudo vi /etc/fstab
```

Add this line at the end:

```
UUID=54cd7e6b-3cf3-443a-876e-1a17362295ec /volume2 xfs defaults,nofail 0 2
```

---

## Step 9: Reboot and Validate

Reboot the instance:

```bash
sudo reboot
```

After reboot, confirm the mount:

```bash
ls /volume2
cat /volume2/text.txt
```

we can see `text.txt` and its content.

---
