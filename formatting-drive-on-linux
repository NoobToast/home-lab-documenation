# Formatting a Drive in Linux Terminal
**Author:** Greg Diny

A practical guide to safely partitioning, formatting, and mounting drives in Linux.

## ⚠️ WARNING
**This will ERASE everything on the target drive!** Double-check you're targeting the correct disk before proceeding.

## Step 1: Identify Your Drives

```bash
lsblk
```

This shows all drives. You should see output like:

```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda           8:0    0    1TB  0 disk 
nvme0n1     259:0    0   256G  0 disk 
├─nvme0n1p1 ...
└─nvme0n1p2 ...
```

**Confirm which drive you want to format** (e.g., `sda` in this example).

## Step 2: Unmount if Mounted

```bash
# Unmount all partitions on the drive
sudo umount /dev/sda* 2>/dev/null
```

## Step 3: Partition the Drive

Using `parted` for clean GPT partitioning:

```bash
# Start parted
sudo parted /dev/sda

# Inside parted, run these commands:
mklabel gpt                    # Create GPT partition table
mkpart primary ext4 0% 100%    # Create one partition using full disk
quit                           # Exit parted
```

## Step 4: Format the Partition

```bash
# Format as ext4 with label "backup"
sudo mkfs.ext4 -L backup /dev/sda1
```

The `-L backup` flag gives the partition a human-readable label.

## Step 5: Create Mount Point and Mount

```bash
# Create mount directory
sudo mkdir -p /mnt/backup

# Mount the partition
sudo mount /dev/sda1 /mnt/backup

# Verify it mounted correctly
df -h | grep sda
```

## Step 6: Auto-Mount on Boot (Recommended)

### Manual Method

1. Get the partition UUID:
```bash
sudo blkid /dev/sda1
```

2. Copy the UUID (looks like: `UUID="abc123-def456..."`)

3. Edit `/etc/fstab`:
```bash
sudo nano /etc/fstab
```

4. Add this line (replace `YOUR-UUID` with actual UUID):
```
UUID=YOUR-UUID /mnt/backup ext4 defaults 0 2
```

5. Save and exit (`Ctrl+X`, `Y`, `Enter`)

6. Test the fstab entry:
```bash
sudo mount -a
```

### Automated Method

Add the drive to fstab automatically:

```bash
echo "UUID=$(sudo blkid -s UUID -o value /dev/sda1) /mnt/backup ext4 defaults 0 2" | sudo tee -a /etc/fstab
```

Verify it was added:

```bash
cat /etc/fstab
```

You should see a new line at the bottom with your UUID and `/mnt/backup`.

## Verify Auto-Mount

```bash
# Unmount first
sudo umount /mnt/backup

# Test that fstab works
sudo mount -a

# Verify it mounted
df -h | grep backup
```

If you see `/mnt/backup` listed with your drive size, it's working! The drive will now auto-mount on every boot.

## Quick Method (All-in-One)

For experienced users, here's the express version:

```bash
# Wipe partition table and create ext4 in one go
sudo wipefs -a /dev/sda
sudo parted /dev/sda mklabel gpt
sudo parted /dev/sda mkpart primary ext4 0% 100%
sudo mkfs.ext4 /dev/sda1
sudo mkdir -p /mnt/backup
sudo mount /dev/sda1 /mnt/backup
echo "UUID=$(sudo blkid -s UUID -o value /dev/sda1) /mnt/backup ext4 defaults 0 2" | sudo tee -a /etc/fstab
```

## Troubleshooting

**Drive won't unmount:**
```bash
# Find what's using it
sudo lsof /dev/sda*

# Kill processes if needed
sudo fuser -km /dev/sda1
```

**fstab entry doesn't work:**
- Verify UUID matches: `sudo blkid /dev/sda1`
- Check for typos in `/etc/fstab`
- Ensure mount point exists: `ls -la /mnt/backup`

**Permission denied:**
- Ensure you're using `sudo` for all commands
- Check if drive is write-protected

---

*This guide reflects practical experience managing Linux systems and storage devices in a home lab environment.*
