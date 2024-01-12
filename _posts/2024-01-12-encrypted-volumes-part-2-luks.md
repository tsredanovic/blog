---
layout: post
title:  Encrypted volumes part 2 - LUKS
categories: [LUKS,Encryption,Disk]
excerpt:  Part 2 of creating an encrypted logical volume using LVM and LUKS with automatic mounting.
---

![]({{site.baseurl}}/images/2024-01-12-encrypted-volumes-part-2-luks.png)

## Encrypted volumes part 2 - LUKS

Linux Unified Key Setup (LUKS) is a disk encryption specification which we will use to encrypt a logical volume created in previous post (part 1).

This part will be pretty straightforward but it will get us to auto mounting (part 3) which is a bit more complicated.

### Creating a passphrase

Let's first create a passphrase which will be used to unlock the encrypted logical volume. This can be done using the `dd` command with the output piped through base64 to get a random string:
```bash
dd bs=32 count=1 if=/dev/random | base64
```
The output should look like this:
```
7DunqeFbznd0SQ9Qwa9GQS0mEFMSeBsoT2zPIzizKOk=
```

This will be your main passphrase so store it somewhere safe and later we will add another passphrase just for auto mounting.

### Encrypting a logical volume

By checking the output of the `lvdisplay` we can see that the `LV Path` of our logical volume is `/dev/vg0/lv0`.

Now we can encrypt it using the `cryptsetup` command:
```bash
cryptsetup luksFormat /dev/vg0/lv0
```

The command will prompt us for confirmation and for the passphrase, use the one generated earlier.

### Create a file system on encrypted logical volume

First we need to open the encrypted logical volume using the `cryptsetup` command:
```bash
cryptsetup open /dev/mapper/vg0-lv0 open-lv0
```

The command will prompt us for passphrase and if entered correctly we can see our open logical volume `open-lv0` using the `ls` command:
```bash
ls /dev/mapper
```

Here is `open-lv0` in the output:
```
control  open-lv0  vg0-lv0
```

Now we can create a file system on the encrypted logical volume using the `mkfs` command:
```bash
mkfs.ext4 /dev/mapper/open-lv0
```

And that's it, we have a perfectly usable encrypted logical volume.

## Useful commands

Some other commands we might find useful like mounting and unmounting manually. These can be used to test things out but won't be needed when we set up auto mounting.

Close an encrypted volume:
```bash
cryptsetup close open-lv0
```

Show information about an encrypted volume:
```bash
cryptsetup luksDump /dev/vg0/lv0
```

Test passphrase at key slot for encrypted volume:
```
cryptsetup luksOpen --test-passphrase --key-slot 0 /dev/vg0/lv0 && echo Correct
```

Mount open logical volume to a folder (`/mnt/lv0` in this example):
```bash
mount /dev/mapper/open-lv0 /mnt/open-lv0
```

Unmount open logical volume:
```bash
umount /dev/mapper/open-lv0
```

