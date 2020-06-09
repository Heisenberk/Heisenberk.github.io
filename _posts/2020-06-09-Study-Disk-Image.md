---
layout: post
title: Study a Disk Image
---

This section explains how to analyze a disk image securely.

There lots of types of disk images. We can list the most recurrent : 
- filesystem data
- DOS/MBR boot sector : it is the first sector addressable in a hard disk
- EWF/Expert Witness/EnCase image file format : contains the contents and structure of a disk volume for example

For example : 

```sh
$ file *
disk1.img: 	DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "mkfs.fat", sectors/cluster 4, root entries 512, Media descriptor 0xf8, sectors/FAT 200, sectors/track 32, heads 64, sectors 204800 (volumes > 32 MB), serial number 0x2b912b13, unlabeled, FAT (16 bit)
disk2.img: 	Linux rev 1.0 ext4 filesystem data, UUID=7bdf2aae-558e-4f2b-86aa-ae5e3b238f1c (extents) (large files) (huge files)
disk3.e01: 	EWF/Expert Witness/EnCase image file format
```

### First step : the Read-Only Mount

To look in a disk image, the first step is mounting the filesystem in read-only to avoid a virus contained in the disk image to spread.

