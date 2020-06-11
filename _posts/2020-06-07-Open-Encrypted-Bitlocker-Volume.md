---
layout: post
title: Open an encrypted Bitlocker volume
---

This section explains how to find the key and open an encrypted Bitlocker volume. 

### Identify a partition encrypted with Bitlocker

```sh
$ fdisk -l image.dd
Disk image.dd: 298.1 GiB, 320072933376 bytes, 625142448 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x51c47769

Device                    Boot     Start       End   Sectors   Size Id Type
image.dd1 *                 2048   1050623   1048576   512M  7 HPFS/NTFS/exFAT
image.dd2                1050624 316475391 315424768 150.4G  7 HPFS/NTFS/exFAT
image.dd3              316475392 625137663 308662272 147.2G  7 HPFS/NTFS/exFAT
```

We have 3 partitions. Moreover, the length of sectors is 512 bytes.

A partition encrypted with Bitlocker can be found with : ```hexdump -C -s $((512*[start_partition])) -n 16 image.dd```

```sh
$ hexdump -C -s $((512*316475392)) -n 16 image.dd
25ba100000  eb 58 90 2d 46 56 45 2d  46 53 2d 00 02 08 00 00  |.X.-FVE-FS-.....|
```
This signature indicates the last partition of ```ìmage.dd``` is a bitlocker volume.

### Find the key 

If you have a live memory dump, you can find a plugin to extract the bitlocker key with Volatility: 

```sh
$ vol -f mem.dmp --profile Win7SP1x64 bitlocker
Volatility Foundation Volatility Framework 2.5

Address : 0xfa8009958c10
Cipher  : AES-256
FVEK    : d5b6e71adb0c2e2d38dafdcedade8fc11e8be631b9fed5b2ba5b51ba32a57cd1
TWEAK   : 49f9ecd5ddffcae44cde7f7a578b9a3ca5e79087826779e147de89423ebdf3f3
```

### Mount a Bitlocker volume

Now, you can mount the bitlocker volume with : 
```sh
$ sudo bdemount -k [FVEK]:[TWEAK] -o $((512*[start_partition]])) image.dd /tmp/
```

For example : 
```sh
$ sudo bdemount -k d5b6e71adb0c2e2d38dafdcedade8fc11e8be631b9fed5b2ba5b51ba32a57cd1:49f9ecd5ddffcae44cde7f7a578b9a3ca5e79087826779e147de89423ebdf3f3 -o $((512*316475392)) image.dd /tmp/
$ mount -t auto /tmp/my_volume mount/
```