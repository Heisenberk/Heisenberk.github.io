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

You can find AES keys (128, 192, 256 bits) in running process memory. A specific tool exists and his name is **<a href="https://sourceforge.net/projects/findaes/" target="_blank">findaes</a>**. 
You can download **<a href="{{ site.baseurl }}/downloads/findaes.zip" target="_blank">here</a>** too. 

```sh
$ ./findaes mem.dmp
Searching mem.dmp
Found AES-128 key schedule at offset 0x1c5f6c30: 
1f ab 01 5c 1e 3d f9 ea c8 72 8f 65 a3 d1 66 46

$ echo 1fab015c1e3df9eac8728f65a3d16646 | xxd -r -p > key.bin
```

#### Extract RSA keys

You can find RSA keys in running process memory. A specific tool exists and his name is **<a href="https://github.com/congwang/rsakeyfind" target="_blank">rsakeyfind</a>**.
You can download **<a href="{{ site.baseurl }}/downloads/rsakeyfind.zip" target="_blank">here</a>** too.

```sh
./rsakeyfind mem.dmp 
FOUND PRIVATE KEY AT c64ac50
version = 
00 
modulus = 
00 d7 1e 77 82 8c 92 31 e7 69 02 a2 d5 5c 78 de 
a2 0c 8f fe 28 59 31 df 40 9c 60 61 06 b9 2f 62 
40 80 76 cb 67 4a b5 59 56 69 17 07 fa f9 4c bd 
6c 37 7a 46 7d 70 a7 67 22 b3 4d 7a 94 c3 ba 4b 
7c 4b a9 32 7c b7 38 95 45 64 a4 05 a8 9f 12 7c 
4e c6 c8 2d 40 06 30 f4 60 a6 91 bb 9b ca 04 79 
11 13 75 f0 ae d3 51 89 c5 74 b9 aa 3f b6 83 e4 
78 6b cd f9 5c 4c 85 ea 52 3b 51 93 fc 14 6b 33 
5d 30 70 fa 50 1b 1b 38 81 13 8d f7 a5 0c c0 8e 
f9 63 52 18 4e a9 f9 f8 5c 5d cd 7a 0d d4 8e 7b 
ee 91 7b ad 7d b4 92 d5 ab 16 3b 0a 8a ce 8e de 
47 1a 17 01 86 7b ab 99 f1 4b 0c 3a 0d 82 47 c1 
91 8c bb 2e 22 9e 49 63 6e 02 c1 c9 3a 9b a5 22 
1b 07 95 d6 10 02 50 fd fd d1 9b be ab c2 c0 74 
d7 ec 00 fb 11 71 cb 7a dc 81 79 9f 86 68 46 63 
82 4d b7 f1 e6 16 6f 42 63 f4 94 a0 ca 33 cc 75 
13 
publicExponent = 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 01 00 01 
privateExponent = 
62 b5 60 31 4f 3f 66 16 c1 60 ac 47 2a ff 6b 69 
00 4a b2 5c e1 50 b9 18 74 a8 e4 dc a8 ec cd 30 
bb c1 c6 e3 c6 ac 20 2a 3e 5e 8b 12 e6 82 08 09 
38 0b ab 7c b3 cc 9c ce 97 67 dd ef 95 40 4e 92 
e2 44 e9 1d c1 14 fd a9 b1 dc 71 9c 46 21 bd 58 
88 6e 22 15 56 c1 ef e0 c9 8d e5 80 3e da 7e 93 
0f 52 f6 f5 c1 91 90 9e 42 49 4f 8d 9c ba 38 83 
e9 33 c2 50 4f ec c2 f0 a8 b7 6e 28 25 56 6b 62 
67 fe 08 f1 56 e5 6f 0e 99 f1 e5 95 7b ef eb 0a 
2c 92 97 57 23 33 36 07 dd fb ae f1 b1 d8 33 b7 
96 71 42 36 c5 a4 a9 19 4b 1b 52 4c 50 69 91 f0 
0e fa 80 37 4b b5 d0 2f b7 44 0d d4 f8 39 8d ab 
71 67 59 05 88 3d eb 48 48 33 88 4e fe f8 27 1b 
d6 55 60 5e 48 b7 6d 9a a8 37 f9 7a de 1b cd 5d 
1a 30 d4 e9 9e 5b 3c 15 f8 9c 1f da d1 86 48 55 
ce 83 ee 8e 51 c7 de 32 12 47 7d 46 b8 35 df 41 
prime1 = 
00 
prime2 = 
00 
exponent1 = 
00 
exponent2 = 
00 
coefficient = 
00 
```

A Write-up is available **<a href="{{ site.baseurl }}/Academy-Investigation-FCSC-2020/" target="_blank">here</a>** to explain key-RSA use.

### Next step 

Now you can study the memory dump with Volatility : firstly, find the correct profile and then, use different commands to understand the content of the memory dump.

