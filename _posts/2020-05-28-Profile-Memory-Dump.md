---
layout: post
title: Identify the correct profile with a live memory dump - Volatility
---

This section explains how to find the profile of a Windows/Linux memory dump with Volatility. 

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
$ strings dump.raw | grep -i 'Linux version' | uniq
Linux version 4.4.0-72-lowlatency (buildd@lcy01-17) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.4) )
```

Now, you can get the identified profile on Github [here](https://github.com/volatilityfoundation/profiles/tree/master/Linux). If you don't find the profile, 
you must create it. 

#### Create a profile for Linux

To create the correct profile, you have to get the same print when you make these commands : 

```sh
$ strings dump.raw | grep -i 'Linux version' | uniq
Linux version 4.4.0-72-lowlatency (buildd@lcy01-17) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.4) )

$ uname -a
Linux version 4.4.0-72-lowlatency (buildd@lcy01-17) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.4) )
```

If you don't have the correct kernel version because you don't find the correct virtual machine, you can upgrade/downgrade your kernel. 

Then, you can make the profile : 

```sh
$ sudo apt install linux-image-4.4.0-72-lowlatency linux-headers-4.4.0-72-lowlatency
$ sudo apt install build-essential dwarfdump
$ git clone --depth=1 https://github.com/volatilityfoundation/volatility
$ cd volatility/tools/linux
$ make
$ sudo zip Ubuntu1604.zip volatility/tools/linux/module.dwarf /boot/System.map-4.4.0-72-lowlatency
$ mv Ubuntu1604.zip /usr/lib/python2.7/dist-packages/volatility/plugins/linux/
```

### Identify the profile for MacOSX

```sh
$ vol -f dump.mem mac_get_profile
Profile                              Shift Address
------------------------------------ -------------
MacSierra_11_8_1_AMDx64              0x00004000000
```

in the same way, you can download the correct profile on Github [here](https://github.com/volatilityfoundation/profiles/tree/master/Mac) or create it. 

### Use a plugin

To use a plugin, you have to specify the plugin. But, sometimes, you download a new plugin. So, to use it : 

```sh
# if you have a Windows plugin
$ sudo cp plugin.py /usr/lib/python2.7/dist-packages/volatility/plugins/
$ volatility -f mem.dmp --profile=Win7SP1x86 plugin

# if you have a Linux plugin
$ sudo cp linux_plugin.py /usr/lib/python2.7/dist-packages/volatility/plugins/linux/
$ volatility -f mem.dmp --profile=LinuxUbuntu1204x64 linux_plugin

# if you have a MacOSX plugin
$ sudo cp mac_plugin.py /usr/lib/python2.7/dist-packages/volatility/plugins/mac/
$ volatility -f mem.dmp --profile=MacMountainLion_10_8_3_AMDx64 mac_plugin
```
