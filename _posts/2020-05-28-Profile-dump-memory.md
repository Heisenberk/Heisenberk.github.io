---
layout: post
title: Identify the correct profile with a memory dump
---

This section explains how find the profile of a memory dump with Volatility. 

In fact, the process is different according to the Operating System (Windows, Linux, MacOSX)

### Identify the profile for Windows 

```sh
$ volatility -f dump.mem imageinfo
Suggested Profile(s) : Win7SP1x86_23418, Win7SP0x86, Win7SP1x86
```
Here, with this command, you determine 3 possible profiles. Then, you can specify the profile with the option ```profile``` : 

```sh
$ volatility -f dump.mem --profile=Win7SP1x86 cmdline
```

### Identify the profile for Linux

```sh
$ strings mem.raw | grep -i 'Linux version' | uniq
Linux version 4.4.0-72-lowlatency (buildd@lcy01-17) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.4) )
```

Now, you can get the identified profile on Github [here](https://github.com/volatilityfoundation/profiles/tree/master/Linux). If you don't find the profile, 
you must create it. 


### Identify the profile for MacOSX

```sh
$ vol -f dump.mem mac_get_profile
Profile                              Shift Address
------------------------------------ -------------
MacSierra_11_8_1_AMDx64              0x00004000000
```

in the same way, you can download the correct profile on Github [here](https://github.com/volatilityfoundation/profiles/tree/master/Mac) or create it. 
