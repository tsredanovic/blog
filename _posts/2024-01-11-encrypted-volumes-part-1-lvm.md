---
layout: post
title:  Encrypted volumes part 1 - LVM
categories: [LVM,Physical,Logical,Disk]
excerpt:  Part 1 of creating an encrypted logical volume using LVM and LUKS with automatic mounting.
---

![]({{site.baseurl}}/images/2024-01-11-encrypted-volumes-part-1-lvm.png)

## Encrypted volumes part 1 - LVM

Logical Volume Manager (LVM) is used to create logical volumes (think of it as virtual disks/partitions) from multiple physical volumes (disk partitions or entire disks).

In this post (part 1) we will focus on creating a single logical volume from multiple hard disks and in the next post (part 2) we will see how to encrypt that logical volume. 

With that in mind there are 3 main aspects of LVM:
1. Physical volumes - hard disks/partitions
2. Volume groups - used for grouping physical volumes
3. Logical volumes - contain defined size available from a volume group

### 1. Physical volumes

Let's start off by listing all disks available on the system using the `fdisk` command and identifing which of them we want to add to our logical volume:
```bash
fdisk -l
```
In this example we have `/dev/sda` and `/dev/sdb`:
```
Disk /dev/sda: 465.76 GiB, 500107862016 bytes, 976773168 sectors
Disk model: nal USB 3.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x4df08fa8

Disk /dev/sdb: 3.64 TiB, 4000752599040 bytes, 7813969920 sectors
Disk model: My Passport 2627
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```

We can remove all partitions from them (replace `sdx` with `sda` and `sdb`):
```bash
fdisk /dev/sdx
```
- delete partiton with `d` command
- verify deletion with `p` command
- confirm with `w` command

And remove any metadata (replace `sdx` with `sda` and `sdb`):
```bash
wipefs --all --backup /dev/sdx
```

Finally mark the disks as LVM physical volumes:
```bash
pvcreate /dev/sda /dev/sdb
```

To verify we can use the `lvmdiskscan` command:
```bash
lvmdiskscan
```
Among everything it shows there should be:
```
  /dev/sda       [     465.76 GiB] LVM physical volume
  /dev/sdb       [      <3.64 TiB] LVM physical volume
```

### 2. Volume groups

To create a volume group simply use the `vgcreate` command, in this example the volume group will be called `vg0` and it will contain `/dev/sda` and `/dev/sdb` physical volumes:
```
vgcreate vg0 /dev/sda /dev/sdb
```
Verify with the `pvs` command:
```bash
pvs
```
We should see both `/dev/sda` and `/dev/sdb` physical volumes in the `vg0` volume group:
```
  PV         VG  Fmt  Attr PSize    PFree
  /dev/sda   vg0 lvm2 a--  <465.76g <465.76g
  /dev/sdb   vg0 lvm2 a--    <3.64t   <3.64t
```

### 3. Logical volumes

Finally we get to creating a logical volume, in this example we will create one named `lv0` with 100% of the available space from our `vg0` volume group:
```bash
lvcreate -l 100%FREE -n lv0 vg0
```

To verify we can use the `lvdisplay` command:
```bash
lvdisplay
```

And there we have it, our `lv0` logical volume with size equal to two of our disks combined:
```
  --- Logical volume ---
  LV Path                /dev/vg0/lv0
  LV Name                lv0
  VG Name                vg0
  LV UUID                BepI42-1dx6-3I0K-fpLV-qUNk-euW5-c3r1GL
  LV Write Access        read/write
  LV Creation host, time server-rpi4, 2024-01-10 13:18:35 +0100
  LV Status              available
  # open                 0
  LV Size                4.09 TiB
  Current LE             1073087
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```

## Useful commands

Some other commands we might find useful, especially if messing up something while following this post:

Delete a logical volume (use `LV Path` from `lvdisplay` command):
```bash
lvremove /dev/vg0/lv0
```

Delete a volume group:
```bash
vgremove vg0
```

Delete a physical volume (replace `sdx` with `sda` or `sdb`):
```bash
pvremove /dev/sdx
```
