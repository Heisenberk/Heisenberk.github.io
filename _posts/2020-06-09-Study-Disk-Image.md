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

### Example : Not So FAT - FCSC 2019

Here, I will explain the resolution of a forensics challenge of FCSC 2019 : 

| Phase          | Catégorie    |   Difficulté  | Nombre de résolutions |
|:--------------:|:------------:|:-------------:|:---------------------:|
| Pré-sélections | forensics    |      Facile   |            324 / 1241 |

J'ai effacé mon flag par erreur, pourriez-vous le retrouver pour moi ?

SHA256(`image.dd`) : `d9a2cdd302bb92a9a7348dc99899c4a5cb5f3b2a75644f4eadbaf3d127def4c7`.

- First step : analyzing what we have : 
```sh
$ sha256sum image.dd
d9a2cdd302bb92a9a7348dc99899c4a5cb5f3b2a75644f4eadbaf3d127def4c7  image.dd
$ file image.dd
image.dd: DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "mkfs.fat", sectors/cluster 4, reserved sectors 4, root entries 512, sectors 32768 (volumes <=32 MB), Media descriptor 0xf8, sectors/FAT 32, sectors/track 32, heads 64, serial number 0x3be84c04, unlabeled, FAT (16 bit)
```
- So, it is a disk image what we could mount : 
```sh
# Mount in Read-Only
$ mkdir mount
$ sudo mount -r image.dd mount/
$ cd mount/
$ ls -lsqa
total 20
16 drwxr-xr-x 2 root    root    16384 Dec 31  1969 .
 4 drwxr-xr-x 3 clement clement  4096 Jun  9 07:55 ..
$ cd ..
$ sudo umount mount
```
- Unfortunately, we don't have files but where are deleted files? 
```sh
$ fls -r image.dd 
r/r * 4:	ziEuYrJW
r/r * 6:	flag.zip
v/v 523203:	$MBR
v/v 523204:	$FAT1
v/v 523205:	$FAT2
V/V 523206:	$OrphanFiles
```
- Yes, there are deleted files. So, we are going to extract them : 
```sh
$ icat image.dd 4 > ziEuYrJW
$ icat image.dd 6 > flag.zip
$ icat image.dd 523203 > MBR
$ icat image.dd 523204 > FAT1
$ icat image.dd 523205 > FAT2
$ icat image.dd 523206 > OrphanFiles
$ file *
FAT1:        ISO-8859 text, with no line terminators
FAT2:        ISO-8859 text, with no line terminators
flag.zip:    Zip archive data, at least v1.0 to extract
image.dd:    DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "mkfs.fat", sectors/cluster 4, reserved sectors 4, root entries 512, sectors 32768 (volumes <=32 MB), Media descriptor 0xf8, sectors/FAT 32, sectors/track 32, heads 64, serial number 0x3be84c04, unlabeled, FAT (16 bit)
MBR:         DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "mkfs.fat", sectors/cluster 4, reserved sectors 4, root entries 512, sectors 32768 (volumes <=32 MB), Media descriptor 0xf8, sectors/FAT 32, sectors/track 32, heads 64, serial number 0x3be84c04, unlabeled, FAT (16 bit)
OrphanFiles: empty
ziEuYrJW:    empty
``` 
- Let's try the string ```ziEuYrJW``` : 
```sh
$ unzip flag.zip 
Archive:  flag.zip
[flag.zip] flag.txt password: 
password incorrect--reenter: 
```
- It is an incorrect password and unfortunately, we don't have any others useful strings. So, we will bruteforce the password : 

```sh
$ fcrackzip -v -u -D -p /usr/share/wordlists/rockyou.txt flag.zip 
found file 'flag.txt', (size cp/uc     59/    47, flags 9, chk 4eb7)


PASSWORD FOUND!!!!: pw == password
$ unzip flag.zip 
Archive:  flag.zip
[flag.zip] flag.txt password: 
 extracting: flag.txt                
$ ls
FAT1  FAT2  flag.txt  flag.zip  image.dd  MBR  OrphanFiles  ziEuYrJW
$ cat flag.txt 
ECSC{eefea8cda693390c7ce0f6da6e388089dd615379}
```

GOTCHA! 
This challenge was very easy but it allows to show how analyze a disk image. 
