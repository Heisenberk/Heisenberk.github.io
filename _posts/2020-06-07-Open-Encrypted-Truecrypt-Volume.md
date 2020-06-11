---
layout: post
title: Open an encrypted Truecrypt volume
---

This section explains how to open an encrypted TrueCrypt volume. 

### Find the key 

If you have a live memory dump, you can find a plugin to extract the truecrypt key with Volatility: 

```sh
volatility -f mem.dmp --profile=Win7SP1x86 truecryptsummary
Volatility Foundation Volatility Framework 2.5
Registry Version     TrueCrypt Version 7.0a
Password             A_Very_Bad_Passw0rd at offset 0x01224426
[...]
```

### Mount a Bitlocker volume

```sh
$ mkdir mount
$ truecrypt volume_bitlocker media/
# here it will prompt for a password (here the password is A_Very_Bad_Passw0rd)
```