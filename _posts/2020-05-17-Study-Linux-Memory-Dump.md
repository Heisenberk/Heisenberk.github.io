---
layout: post
title: Study a live Linux memory dump - Volatility
---

This section explains the main commands in Volatility to analyze a Linux memory dump. 

### Linux Processes

- See processes : 

```sh
$ volatility -f mem.dmp --profile=LinuxUbuntu1204x64 linux_pslist
Volatility Foundation Volatility Framework 2.4
Offset             Name                 Pid             Uid             Gid    DTB                Start Time
------------------ -------------------- --------------- --------------- ------ ------------------ ----------
0xffff88007b818000 init                 1               0               0      0x00000000366ec000 Fri, 17 Aug 2012 19:55:38 +0000
0xffff88007b8196f0 kthreadd             2               0               0      ------------------ Fri, 17 Aug 2012 19:55:38 +0000
0xffff88007b81ade0 ksoftirqd/0          3               0               0      ------------------ Fri, 17 Aug 2012 19:55:38 +0000
0xffff88007b81c4d0 kworker/0:0          4               0               0      ------------------ Fri, 17 Aug 2012 19:55:38 +0000
[snip]
0xffff8800790c5bc0 gnome-pty-helpe      11285           1000            1000   0x00000000308c1000 Fri, 17 Aug 2012 21:29:31 +0000
0xffff88007ad15bc0 bash                 11286           1000            1000   0x00000000309fa000 Fri, 17 Aug 2012 21:29:31 +0000
0xffff88005b8bdbc0 firefox              11370           1000            1000   0x00000000308a8000 Fri, 17 Aug 2012 21:31:22 +0000
0xffff880079f62de0 at-spi-bus-laun      11389           1000            1000   0x0000000030b8b000 Fri, 17 Aug 2012 21:31:22 +0000
0xffff880027d28000 notify-osd           18366           1000            1000   0x0000000027d10000 Fri, 17 Aug 2012 22:30:37 +0000
0xffff88005b8c16f0 kworker/0:1          18535           0               0      ------------------ Fri, 17 Aug 2012 22:31:13 +0000
0xffff880065ac44d0 kworker/0:2          18646           0               0      ------------------ Fri, 17 Aug 2012 22:36:14 +0000
0xffff880030b22de0 sudo                 18649           1000            1000   0x0000000027ed3000 Fri, 17 Aug 2012 22:36:42 +0000
0xffff880027efc4d0 insmod               18650           0               0      0x00000000309db000 Fri, 17 Aug 2012 22:36:42 +0000
```

- See the equivalent of ```ps aux``` : 

```sh
$ volatility -f mem.dmp --profile=LinuxUbuntu1204x64 linux_psaux
Volatility Foundation Volatility Framework 2.4
Pid    Uid    Arguments                                                       
1      0      /sbin/init ro quiet splash                                       Fri, 17 Aug 2012 19:55:38 +0000    
2      0      [kthreadd]                                                       Fri, 17 Aug 2012 19:55:38 +0000    
3      0      [ksoftirqd/0]                                                    Fri, 17 Aug 2012 19:55:38 +0000    
4      0      [kworker/0:0]                                                    Fri, 17 Aug 2012 19:55:38 +0000
[snip]
11370  1000   /usr/lib/firefox/firefox                                         Fri, 17 Aug 2012 21:31:22 +0000    
11389  1000   /usr/lib/x86_64-linux-gnu/at-spi2-core/at-spi-bus-launcher       Fri, 17 Aug 2012 21:31:22 +0000    
18366  1000   /usr/lib/notify-osd/notify-osd                                   Fri, 17 Aug 2012 22:30:37 +0000    
18535  0      [kworker/0:1]                                                    Fri, 17 Aug 2012 22:31:13 +0000    
18646  0      [kworker/0:2]                                                    Fri, 17 Aug 2012 22:36:14 +0000    
18649  1000   sudo insmod lime-3.2.0-23-generic.ko path=/home/mhl/ubuntu.lime format=lime  Fri, 17 Aug 2012 22:36:42 +0000    
18650  0      insmod lime-3.2.0-23-generic.ko path=/home/mhl/ubuntu.lime format=lime  Fri, 17 Aug 2012 22:36:42 +0000
```

- See the tree of processes :

```sh
$ volatility -f mem.dmp --profile=LinuxUbuntu1204x64 linux_pstree
Volatility Foundation Volatility Framework 2.4
Name                 Pid             Uid            
init                 1               0              
.upstart-udev-br     375             0              
.udevd               412             0              
..udevd              9052            0              
..udevd              9053            0              
.upstart-socket-     707             0          
[snip]
.unity-2d-spread     11236           1000           
.gnome-control-c     11244           1000           
.gnome-terminal      11279           1000           
..gnome-pty-helpe    11285           1000           
..bash               11286           1000           
...sudo              18649           1000           
....insmod           18650           0              
.firefox             11370           1000 
[snip]
```

With this command, you can see suspicious processes. 

- see open files for a specific process : 

```sh
$ volatility -f mem.dmp --profile=LinuxUbuntu1204x64 linux_lsof --pid 1
Volatility Foundation Volatility Framework 2.4
Pid      FD       Path
-------- -------- ----
       1        0 /dev/null
       1        1 /dev/null
       1        2 /dev/null
       1        3 /
       1        4 /
       1        5 inotify
       1        6 inotify
       1        7 /
       1        8 /
       1        9 /
       1       10 /var/log/upstart/modemmanager.log
       1       11 /
       1       18 /dev/ptmx
       1       19 /dev/ptmx
```

You can delete the option ```-p/--pid``` to discover all opened files.

- Print current commands : 

```sh
$ python vol.py -f avgcoder.mem --profile=LinuxCentOS63x64 linux_bash
Volatility Foundation Volatility Framework 2.3_alpha
Pid      Name                 Command Time                   Command
-------- -------------------- ------------------------------ -------
    2738 bash                 2013-08-09 21:28:13 UTC+0000   dmesg | head -50
    2738 bash                 2013-08-09 21:51:28 UTC+0000   df
    2738 bash                 2013-08-09 21:51:50 UTC+0000   dmesg | tail -50
    2738 bash                 2013-08-09 21:51:58 UTC+0000   sudo mount /dev/sda1 /mnt
    2738 bash                 2013-08-09 21:52:02 UTC+0000   cd /mnt
    2738 bash                 2013-08-09 21:52:02 UTC+0000   ls
    2738 bash                 2013-08-09 21:52:08 UTC+0000   sudo insmod rootkit.ko
    2738 bash                 2013-08-09 21:52:56 UTC+0000   echo "hide" > /proc/buddyinfo 
    2738 bash                 2013-08-09 21:53:00 UTC+0000   lsmod | grep root
    2738 bash                 2013-08-09 21:53:14 UTC+0000   w
    2738 bash                 2013-08-09 21:53:38 UTC+0000   echo "huser centoslive" > /proc/buddyinfo 
    2738 bash                 2013-08-09 21:53:40 UTC+0000   w
    2738 bash                 2013-08-09 21:53:49 UTC+0000   sleep 900 &
    2738 bash                 2013-08-09 21:54:01 UTC+0000   echo "hpid 2872" > /proc/buddyinfo 
    2738 bash                 2013-08-09 21:54:13 UTC+0000   ps auwx | grep sleep
    2738 bash                 2013-08-09 21:54:01 UTC+0000   echo "hpid 2872" > /proc/buddyinfo 
    2738 bash                 2013-08-09 21:54:13 UTC+0000   ?
    2738 bash                 2013-08-09 21:52:08 UTC+0000   sudo insmod rootkit.ko
```

### Linux files

- Extract cached files : 

```sh
$ volatility -f mem.dmp --profile=LinuxCentOS63x64 linux_find_file -F "/var/run/utmp"
Volatility Foundation Volatility Framework 2.2_rc1
Inode Number                  Inode
---------------- ------------------
          130564     0x88007a85acc0

$ python -f mem.dmp --profile=LinuxCentOS63x64 linux_find_file -i 0x88007a85acc0 -O utmp
```

### Linux networking

- View active connections : 

```sh
$ volatility --profile=LinuxDebianx86 -f mem.dmp linux_netstat -p 2777
Volatility Foundation Volatility Framework 2.2_rc1
TCP      192.168.110.150:13377 192.168.110.140:41744 CLOSE_WAIT           _h4x_bd/2777
TCP      0.0.0.0:13377         0.0.0.0:0             LISTEN                       _h4x_bd/2777
TCP      192.168.110.150:13377 192.168.110.140:41745 ESTABLISHED           _h4x_bd/2777
[snip]
```

- Find network packets that are in kernel memory

```sh
$ volatility --profile=LinuxDebianx86 -f mem.dmp linux_sk_buff_cache -D recovered_packets
Volatility Foundation Volatility Framework 2.2_rc1
Wrote 20 bytes to de2c60c0
Wrote 1430 bytes to de2da900
Wrote 60 bytes to de21c680
Wrote 42 bytes to de2cc600
Wrote 1430 bytes to de284f00
Wrote 68 bytes to def720c0
Wrote 68 bytes to def72540

$ strings recovered_packets/*
<snip>
```

For more information, check this **<a href="https://github.com/volatilityfoundation/volatility/wiki/Linux-Command-Reference" target="_blank">link</a>**
