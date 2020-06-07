---
layout: post
title: Open a LUKS encrypted file
---

This section explains how to open a LUKS encrypted file. 

```sh
backup/forensic.img: LUKS encrypted file, ver 1 [aes, ecb, sha1] UUID: f832c51b-4f4d-446d-9d21-746b5c26095b
```

### If you get the LUKS-passphrase

```sh
$ mkdir mount1
$ sudo cryptsetup luksOpen mem.img mount1
Enter passphrase for mem.img: 
$ sudo mount /dev/mapper/mount1 mount1/
$ cd mount1/
$ [...]
$ cd ..
$ sudo umount mount1
```

### Else

If you have a live memory dump, you can extract the key with ```findaes``` (you can find the post **Study a live memory dump <a href="{{ site.baseurl }}/Study-Memory-Dump/" target="_blank">here</a>**) : 

```sh
$ ./findaes live-mem.dmp
Searching live-mem.dmp
Found AES-128 key schedule at offset 0x1c5f6c30: 
1f ab 01 5c 1e 3d f9 ea c8 72 8f 65 a3 d1 66 46

$ echo 1fab015c1e3df9eac8728f65a3d16646 | xxd -r -p > key.bin

$ mkdir mount2
$ sudo cryptsetup --master-key-file key.bin luksOpen mem.img mount2
$ sudo mount /dev/mapper/mount2 mount2
$ cd mount2/
$ [...]
$ cd ..
$ sudo umount mount2
```