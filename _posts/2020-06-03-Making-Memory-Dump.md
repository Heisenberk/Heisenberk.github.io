---
layout: post
title: Make a live memory dump to analyze it - Volatility
---

This section explains how to make a memory dump on Windows and Linux. 

### Make a memory dump on Windows

With **DumpIt (you can find it <a href="{{ site.baseurl }}/downloads/DumpIt.zip" target="_blank">here</a>)** : 

```sh
$ DumpIt.exe
```

Then , you can analyse the result with volatility.

### Make a memory dump on Linux

```sh
$ git clone https://github.com/504ensicsLabs/LiME
$ cd LiME/src
$ make
$ cd ~
$ sudo insmod ./LiME/src/lime-XXX.ko path=./mem.dmp format=lime
$ ls 
LiME	mem.dmp
```

Then, you can analyse the result with volatility. Do not forget to determine the correct profile before.

