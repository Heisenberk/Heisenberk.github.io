---
layout: post
title: Study a live MacOSX memory dump - Volatility
---

This section explains the main commands in Volatility to analyze a MacOSX memory dump. 

### MacOSX Processes

- See processes : 

```sh
$ volatility --profile=MacMountainLion_10_8_3_AMDx64 -f mem.dmp mac_pslist
Volatility Foundation Volatility Framework 2.4
Offset             Name                 Pid      Uid      Gid      PGID     Bits         DTB                Start Time
------------------ -------------------- -------- -------- -------- -------- ------------ ------------------ ----------
0xffffff8032be4ea0 image                4175     0        0        4167     64BIT        0x0000000317e7e000 2013-03-29 12:16:20 UTC+0000
0xffffff803dfdea40 coresymbolicatio     4173     0        0        4173     64BIT        0x00000004114c0000 2013-03-29 12:16:18 UTC+0000
0xffffff8032498d20 MacMemoryReader      4168     0        0        4167     64BIT        0x00000003f94a8000 2013-03-29 12:16:17 UTC+0000
0xffffff803dfe0020 sudo                 4167     0        20       4167     64BIT        0x0000000414a34000 2013-03-29 12:16:15 UTC+0000
0xffffff803dfe1a60 mdworker             4164     89       89       4164     64BIT        0x00000003f70cf000 2013-03-29 12:15:32 UTC+0000
0xffffff80370af760 DashboardClient      4160     501      20       275      64BIT        0x00000003e5bd9000 2013-03-29 12:14:36 UTC+0000
0xffffff803634ba60 CVMCompiler          4127     501      20       4127     64BIT        0x000000016692b000 2013-03-29 12:10:58 UTC+0000
0xffffff80370b11a0 cookied              4126     501      20       4126     64BIT        0x00000003137cc000 2013-03-29 12:10:58 UTC+0000
0xffffff803dfe1600 WebProcess           4124     501      20       4121     64BIT        0x00000003f235a000 2013-03-29 12:10:57 UTC+0000
0xffffff803249c600 taskgated            4122     0        0        4122     64BIT        0x00000003f3038000 2013-03-29 12:10:57 UTC+0000
0xffffff80314a9d40 Safari               4121     501      20       4121     64BIT        0x00000003f616c000 2013-03-29 12:10:57 UTC+0000
[snip]
```

- See the equivalent of ```ps aux``` : 

```sh
$ volatility --profile=MacMountainLion_10_8_3_AMDx64 -f mem.dmp mac_psaux
Volatility Foundation Volatility Framework 2.4
Pid      Name                 Bits             Stack              Length   Argc     Arguments
-------- -------------------- ---------------- ------------------ -------- -------- ---------
       0 kernel_task          64BIT            0x0000000000000000        0        0 
[snip]
      40 mDNSResponder        64BIT            0x00007fff54403000      384        2 /usr/sbin/mDNSResponder -launchd
      41 networkd             64BIT            0x00007fff50d3f000      360        1 /usr/libexec/networkd
      59 warmd                64BIT            0x00007fff51816000      232        1 /usr/libexec/warmd
      60 usbmuxd              64BIT            0x00007fff5fc00000      504        2 /System/Library/PrivateFrameworks/MobileDevice.framework/Versions/A/Resources/usbmuxd -launchd
      63 stackshot            64BIT            0x00007fff59a7a000      224        2 /usr/libexec/stackshot -t
      64 SleepServicesD       64BIT            0x00007fff582d5000      280        1 /System/Library/CoreServices/SleepServicesD
      66 revisiond            64BIT            0x00007fff50d1b000      376        1 /System/Library/PrivateFrameworks/GenerationalStorage.framework/Versions/A/Support/revisiond
      71 netbiosd             64BIT            0x00007fff577d3000      360        1 /usr/sbin/netbiosd
      72 mds                  64BIT            0x00007fff5713e000      376        1 /System/Library/Frameworks/CoreServices.framework/Frameworks/Metadata.framework/Support/mds
      75 loginwindow          64BIT            0x00007fff59635000      328        2 /System/Library/CoreServices/loginwindow.app/Contents/MacOS/loginwindow console
      77 KernelEventAgent     64BIT            0x00007fff5e5b7000      232        1 /usr/sbin/KernelEventAgent
      78 kdc                  64BIT            0x00007fff54a32000      304        1 /System/Library/PrivateFrameworks/Heimdal.framework/Helpers/kdc
      80 hidd                 64BIT            0x00007fff56819000      232        1 /usr/libexec/hidd
      82 dynamic_pager        64BIT            0x00007fff5dfdd000      248        3 /sbin/dynamic_pager -F /private/var/vm/swapfile
      84 dpd                  64BIT            0x00007fff5f995000      232        1 /usr/libexec/dpd
      85 corestoraged         64BIT            0x00007fff5dfed000      232        1 /usr/libexec/corestoraged
      86 appleeventsd         64BIT            0x00007fff5cc55000      408        2 /System/Library/CoreServices/appleeventsd --server
      89 blued                64BIT            0x00007fff5a65d000      224        1 /usr/sbin/blued
      91 autofsd              64BIT            0x00007fff577d9000      208        1 /usr/libexec/autofsd autofsd
      95 ntpd                 64BIT            0x00007fff5a494000      296        9 /usr/sbin/ntpd -c /private/etc/ntp-restrict.conf -n -g -p /var/run/ntpd.pid -f /var/db/ntp.drift
[snip]
```

- See the tree of processes :

```sh
$ volatility --profile=MacMountainLion_10_8_3_AMDx64 -f mem.dmp mac_pstree
Volatility Foundation Volatility Framework 2.4
Name                 Pid             Uid            
kernel_task          0               0              
.launchd             1               0              
..coresymbolicatio   4173            0              
..taskgated          4122            0              
..ocspd              973             0              
..launchd            561             89             
...mdworker          4164            89             
...cfprefsd          566             89             
...distnoted         565             89             
..VDCAssistant       558             0              
..Dropbox            518             501            
...dbfseventsd       545             0              
....dbfseventsd      546             0              
.....dbfseventsd     552             501            
.....dbfseventsd     549             501            
..vmware-usbarbitr   461             0     
[snip]
```

With this command, you can see suspicious processes. 

- see open files : 

```sh
$ volatility --profile=MacMountainLion_10_8_3_AMDx64 -f mem.dmp mac_lsof
Volatility Foundation Volatility Framework 2.4
0 -> /Macintosh HD/dev/null
1 -> /Macintosh HD/dev/null
2 -> /Macintosh HD/dev/null
4 -> /Macintosh HD/dev/console
81 -> /Macintosh HD/dev/autofs_nowait
0 -> /Macintosh HD/dev/null
1 -> /Macintosh HD/dev/null
2 -> /Macintosh HD/dev/null
[snip]
19 -> /Macintosh HD/Users/michaelligh/Desktop/volatility/volatility/plugins/mac/pstasks.py
20 -> /Macintosh HD/Users/michaelligh/Desktop/volatility/volatility/plugins/mac/pstree.py
21 -> /Macintosh HD/Users/michaelligh/Desktop/volatility/volatility/plugins/mac/pgrp_hash_table.py
22 -> /Macintosh HD/Users/michaelligh/Desktop/volatility/volatility/plugins/mac/pslist.py
23 -> /Macintosh HD/Users/michaelligh/Desktop/volatility/volatility/plugins/mac/psaux.py
24 -> /Macintosh HD/Users/michaelligh/Desktop/2.3.todo.txt
25 -> /Macintosh HD/Users/michaelligh/Desktop/mac_profile.sh
[snip]
```

- Show the allocated memory blocks in a specific process : 

```sh
$ volatility --profile=MacMountainLion_10_8_3_AMDx64 -f mem.dmp mac_proc_maps -p 1 
Volatility Foundation Volatility Framework 2.4
Pid      Name                 Start              End                Perms     Map Name
-------- -------------------- ------------------ ------------------ --------- --------
1        launchd              0x000000010630c000 0x0000000106333000 r-x       Macintosh HD/sbin/launchd
1        launchd              0x0000000106333000 0x0000000106335000 rw-       Macintosh HD/sbin/launchd
1        launchd              0x0000000106335000 0x000000010633b000 r--       Macintosh HD/sbin/launchd
1        launchd              0x000000010633b000 0x000000010633c000 r--       
1        launchd              0x000000010633c000 0x000000010633f000 r-x       Macintosh HD/usr/lib/libauditd.0.dylib
1        launchd              0x000000010633f000 0x0000000106340000 rw-       Macintosh HD/usr/lib/libauditd.0.dylib
1        launchd              0x0000000106340000 0x0000000106343000 r--       Macintosh HD/usr/lib/libauditd.0.dylib
1        launchd              0x0000000106343000 0x0000000106344000 r--       
1        launchd              0x0000000106344000 0x0000000106345000 rw-       Macintosh HD/private/var/db/dyld/dyld_shared_cache_x86_64
[snip]
```
- Dump memory blocks :

```sh
$ volatility --profile=MacMountainLion_10_8_3_AMDx64 -f mem.dmp -p 1 -s 0x000000010630c000 -O launchd.binary.dmp
Volatility Foundation Volatility Framework 2.4
Wrote 159744 bytes

$ file launchd.binary.dmp 
launchd.binary.dmp: Mach-O 64-bit executable x86_64
```

### Linux networking

- View active connections : 

```sh
$ volatility --profile=MacMountainLion_10_8_3_AMDx64 -f mem.dmp mac_netstat
Volatility Foundation Volatility Framework 2.4
UNIX -
UNIX /var/tmp/launchd/sock
UNIX -
UNIX /var/tmp/com.barebones.authd.socket
UNIX /var/run/com.apple.ActivityMonitor.socket
TCP :::548 :::0 TIME_WAIT
TCP 0.0.0.0:548 0.0.0.0:0 TIME_WAIT
UDP 127.0.0.1:60762 0.0.0.0:0 
UNIX /var/run/mDNSResponder
UNIX /var/rpc/ncacn_np/lsarpc
UNIX /var/rpc/ncalrpc/lsarpc
TCP 10.0.1.3:49179 173.194.76.125:5222 TIME_WAIT
TCP 10.0.1.3:49188 205.188.248.150:443 TIME_WAIT
TCP 10.0.1.3:49189 205.188.254.208:443 TIME_WAIT
TCP 10.0.1.3:50614 205.188.13.76:443 TIME_WAIT
UDP 0.0.0.0:137 0.0.0.0:0 
UDP 0.0.0.0:138 0.0.0.0:0 
UNIX /var/run/vpncontrol.sock
UNIX /var/run/portmap.socket
TCP :::5900 :::0 TIME_WAIT
[snip]
```

- Show routing table : 

```sh
$ volatility --profile=MacMountainLion_10_8_3_AMDx64 -f mem.dmp mac_route
Volatility Foundation Volatility Framework 2.4
Source IP                Dest. IP                    Name           Sent               Recv                     Time                 Exp.    Delta
------------------------ ------------------------ ---------- ------------------ ------------------ ------------------------------ ---------- -----
0.0.0.0                  10.0.1.1                    en1            4342              50431         2013-03-29 01:08:55 UTC+0000      0      0
10.0.1.0                                             en1            8331              31691         2013-03-29 01:08:56 UTC+0000      8      0
10.0.1.1                 00:26:bb:6c:8e:64           en1            4551               4517         2013-03-29 01:08:53 UTC+0000    40318    40310
10.0.1.2                 ac:16:2d:32:fc:d7           en1             1                  47          2013-03-29 11:56:02 UTC+0000    40037    1201
10.0.1.3                 127.0.0.1                   lo0             0                 6168         2013-03-29 01:08:55 UTC+0000      0      0
10.0.1.8                 e8:8d:28:cb:67:07           en1             19                924          2013-03-29 11:56:30 UTC+0000    40065    1201
10.0.1.255               ff:ff:ff:ff:ff:ff           en1             12                 0           2013-03-29 12:13:59 UTC+0000    39913    0
17.171.4.15              10.0.1.1                    en1             39                 39          2013-03-29 01:08:55 UTC+0000      0      0
17.172.232.105           10.0.1.1                    en1             2                  60          2013-03-29 01:09:16 UTC+0000      0      0
17.172.238.203           10.0.1.1                    en1             0                  58          2013-03-29 01:09:46 UTC+0000      0      0
[snip]
```

For more information, check this **<a href="https://github.com/volatilityfoundation/volatility/wiki/Mac-Command-Reference" target="_blank">link</a>**
