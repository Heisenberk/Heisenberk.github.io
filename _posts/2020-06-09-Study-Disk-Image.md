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

### Mount in Read-Only

To look in a disk image, the first step is mounting the filesystem in read-only to avoid a virus contained in the disk image to spread. Then, you can analyze it like a normal filesystem :

```sh
$ mkdir mount
$ sudo mount -r disk.img mount/
$ cd disk2/
$ [...]
$ cd ..
$ sudo umount disk2
```

For a EWF/expert Witness/EnCase image file, you can not use ```mount```. The only way is : 

```sh
$ mkdir mount
$ ewfmount disk3.e01 mount/
```

### Extract deleted files

- You can list files and folders with ```fls```. Deleted files are mentioned by a ```*```

```sh
$ fls -r disk1.img 
r/r 5:	Document confidentiel.docx
r/r * 7:	virus.exe
v/v 3270387:	$MBR
v/v 3270388:	$FAT1
v/v 3270389:	$FAT2
V/V 3270390:	$OrphanFiles
```
So You can extract files with ```icat``` : 

```sh
$ icat disk1.img 7 > virus_extraction.exe
```

- To have the list of deleted files : 

```sh
fls -d -p disk1.img
```

- To see all inodes : 

```sh
$ ils disk1.img -a
```

A Write-up is available **<a href="{{ site.baseurl }}/Not-So-FAT-FCSC-2019/" target="_blank">here</a>** to explain an use.
