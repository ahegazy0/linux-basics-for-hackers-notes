# Linux Basics for Hackers
## Module 10 — Filesystem & Storage

---

## What this module is about

Linux has a completely different relationship with storage devices than Windows does. No drive letters, no automatic pop-ups when you plug something in. Everything is a file, every device lives somewhere in the filesystem tree, and if you want to use a drive you have to explicitly attach it. This module explains how all of that works.

---

## Everything is a file

In Linux, the phrase "everything is a file" isn't just a saying — it's literally how the system is built. Your hard drive isn't a separate thing that exists outside the filesystem. It's represented as a file inside `/dev`. Your keyboard is a file. Your network card is a file. Devices are just special files that the kernel knows how to talk to.

Storage devices live in `/dev` and are named like this:

| Device | What it is |
|---|---|
| `/dev/sda` | First hard drive |
| `/dev/sdb` | Second hard drive |
| `/dev/sdc` | Third hard drive (or USB) |
| `/dev/sda1` | First partition on the first drive |
| `/dev/sda2` | Second partition on the first drive |
| `/dev/sr0` | CD/DVD drive |

The `sd` stands for SCSI disk — the naming convention comes from an older interface standard, but it's used for modern SATA and USB drives too. The letter after `sd` is the drive order (`a` = first, `b` = second). The number after that is the partition number.

So `/dev/sdb3` means: second drive, third partition.

---

## Mounting — how Linux attaches drives

In Windows, plug in a USB and it automatically gets a drive letter like `D:`. Linux doesn't do that. You have to **mount** the drive yourself.

Mounting means telling Linux: "attach this device to this folder in the tree." Once it's mounted, you access the drive by navigating to that folder. The folder it gets attached to is called the **mount point**.

```
/
└── mnt/
    └── usb/    ← your USB drive appears here after mounting
```

The standard locations for mount points are `/mnt` (for temporary manual mounts) and `/media` (where the system auto-mounts things like USB drives in desktop environments).

**To mount a drive:**

```bash
mount /dev/sdb1 /mnt/usb
```

This attaches the first partition of your second drive to the folder `/mnt/usb`. After this, going into `/mnt/usb` shows you the files on that drive.

The folder must already exist before you mount to it:

```bash
mkdir /mnt/usb
mount /dev/sdb1 /mnt/usb
```

**To unmount:**

```bash
umount /mnt/usb
```

Note the spelling — it's `umount`, not `unmount`. Always unmount before physically unplugging a drive. If the system is still writing data to it when you yank it out, you can corrupt the filesystem.

---

## fdisk — seeing what drives you have

`fdisk -l` lists every drive and every partition currently connected to the system. It requires root.

```bash
fdisk -l
```

Output looks something like:

```
Disk /dev/sda: 500 GB, 500107862016 bytes
Device     Boot    Start      End  Sectors  Size  Type
/dev/sda1  *        2048  1026047  1024000  500M  EFI System
/dev/sda2        1026048 97654784 96628737 46.1G  Linux filesystem

Disk /dev/sdb: 16 GB, 16013942784 bytes
Device     Boot  Start     End  Sectors  Size  Type
/dev/sdb1  *      2048 31252479 31250432 14.9G  Microsoft basic data
```

From this you can see: two drives connected, what partitions each has, the size of each partition, and the filesystem type. This is your first step whenever you're dealing with an unknown machine — you want to know what storage is attached.

---

## df — how much space is left

`df` stands for "disk free." It shows how much space is used and available on every mounted filesystem.

```bash
df -h
```

The `-h` flag means human-readable — shows sizes in GB and MB instead of raw bytes.

Output:

```
Filesystem      Size  Used Avail Use%  Mounted on
/dev/sda2        46G   12G   32G  27%  /
/dev/sdb1        15G  8.1G  6.9G  54%  /mnt/usb
tmpfs           2.0G  1.2M  2.0G   1%  /run
```

The `Use%` column tells you at a glance which partitions are filling up. `tmpfs` entries are virtual filesystems in RAM — not actual disk space.

---

## /etc/fstab — the persistent mount config

When you mount something with the `mount` command, it's temporary. After a reboot, the drive is no longer mounted. To make a mount permanent — to have it automatically mount at boot — you add an entry to `/etc/fstab`.

```bash
cat /etc/fstab
```

Each line in fstab describes one filesystem to mount:

```
# device          mountpoint    fstype   options    dump  pass
/dev/sda1         /             ext4     defaults    0     1
/dev/sda2         /home         ext4     defaults    0     2
UUID=1234-ABCD    /mnt/data     ntfs     defaults    0     0
```

From a recon perspective, `/etc/fstab` tells you a lot about a target system. It shows every drive that gets mounted at boot — including network shares (NFS, SMB), encrypted volumes, and external disks. Hidden or unusual entries here can point to things worth investigating.

---

## fsck — checking and repairing drives

`fsck` stands for "filesystem check." It scans a drive for errors and can attempt to fix them.

```bash
fsck /dev/sdb1
```

**The critical rule: never run fsck on a mounted filesystem.** If the drive is mounted and in use, fsck can make the corruption worse, not better. Always unmount first:

```bash
umount /dev/sdb1
fsck /dev/sdb1
```

If you're checking your root partition (`/`), you can't unmount it while the system is running — you'd need to boot from a live USB and run fsck from there.

---

## Filesystem types

When you format a partition, you choose a filesystem type. Different operating systems use different formats. Knowing these matters when you're mounting drives from other systems.

| Filesystem | Where you see it |
|---|---|
| ext4 | Standard Linux filesystem |
| ext3 / ext2 | Older Linux filesystems |
| NTFS | Windows drives |
| FAT32 / exFAT | USB drives, SD cards (cross-platform) |
| XFS | High-performance Linux, often on servers |

When mounting a non-Linux drive, you may need to specify the type:

```bash
mount -t ntfs /dev/sdb1 /mnt/windows
```

Linux can read NTFS drives fine. Writing to them works too but historically had some quirks — just worth knowing.

---

## Mounting a USB drive — the full workflow

This is the complete process from plug-in to accessing files:

```bash
# 1. See what just got connected
fdisk -l

# 2. Create a mount point if it doesn't exist
mkdir /mnt/usb

# 3. Mount the partition
mount /dev/sdb1 /mnt/usb

# 4. Check it worked
ls /mnt/usb

# 5. Do whatever you need to do

# 6. Unmount when done
umount /mnt/usb
```

---

## Command reference

| Command | What it does |
|---|---|
| `fdisk -l` | List all drives and partitions |
| `mount /dev/sdb1 /mnt/point` | Mount a partition to a folder |
| `mount -t ntfs /dev/sdb1 /mnt/point` | Mount with a specific filesystem type |
| `umount /mnt/point` | Safely unmount a drive |
| `df -h` | Show disk space usage (human-readable) |
| `fsck /dev/sdb1` | Check and repair a filesystem (unmount first) |
| `cat /etc/fstab` | Show permanent mount configuration |
| `mkdir /mnt/point` | Create a mount point directory |
| `lsblk` | Clean tree view of drives and partitions |

One extra worth knowing: `lsblk` gives you a cleaner view than `fdisk -l` for just seeing your drive layout:

```bash
lsblk
```

Output:
```
NAME   MAJ:MIN  SIZE  TYPE  MOUNTPOINT
sda    8:0      500G  disk
├─sda1 8:1      500M  part  /boot
└─sda2 8:2      499G  part  /
sdb    8:16      16G  disk
└─sdb1 8:17     16G  part  /mnt/usb
```

Much easier to read at a glance than raw fdisk output.

---

## Practice

- Run `fdisk -l` and identify every drive and partition connected to your VM
- Run `df -h` and find which partition is closest to full
- Create `/mnt/test` with `mkdir`, plug in a USB drive (or add a virtual disk in VirtualBox), find its device name with `fdisk -l`, and mount it to `/mnt/test`
- After mounting, run `ls /mnt/test` to confirm it worked, then `umount /mnt/test` when done
- Read through `/etc/fstab` and understand what each line is doing

---

*Up next: Module 11 — Logging & Log Files*
