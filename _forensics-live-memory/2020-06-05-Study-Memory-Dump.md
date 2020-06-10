---
layout: post
title: Study a live memory dump
---

This section explains how to analyze a memory dump before using Volatility : extracting files and secrets.

Before analyzing a memory dump with Volatility, you can conduct a little background research : 

### Live Memory Dump Format

The format of a memory dump can be : 
- full data 
- a core file 

```sh
$ file mem1.dmp
mem1.dmp:	data
$file mem2.dmp
mem2.dmp:	ELF 64-bit LSB core file, x86-64, version 1 (SYSV)	
```

### Extract files

- extract a pcap file which represents packets passing through the machine : 

```sh
$ bulk_extractor -x all -e net -o mem.dmp
```

- extract (specific) files : 

```sh
$ foremost -o result/ -t zip -i mem.dmp
$ binwalk --dd='.*' mem.dmp
```

In these examples, foremost extracts zip files and binwalk extracts all files in the memory dump.

### Extract information

- use strings and grep to determine some simple information : 

```sh
strings mem.dmp | grep -B 10 -A 20 "some text"
```

This example prints results with 10 lines before the result and 20 lines after the result.

```sh
strings mem.dmp | grep -i "text"
```

This example use the option ```-i``` to perform a case-insensitive match.

With these techniques, you can search specific words like **passphrase**, **password**, **pass**, **phrase**, ...

### Extract secrets

#### Extract AES keys

You can find AES keys (128, 192, 256 bits) in running process memory. A specific tool can be found **<a href="https://sourceforge.net/projects/findaes/" target="_blank">here</a>**. 
You can download **<a href="{{ site.baseurl }}/downloads/findaes.zip" target="_blank">here</a>** too. 

```sh
$ ./findaes mem.dmp
Searching mem.dmp
Found AES-128 key schedule at offset 0x1c5f6c30: 
1f ab 01 5c 1e 3d f9 ea c8 72 8f 65 a3 d1 66 46

$ echo 1fab015c1e3df9eac8728f65a3d16646 | xxd -r -p > key.bin
```

### Next step 

Now you can study the memory dump with Volatility : firstly, find the correct profile and then, use different commands to understand the content of the memory dump.
