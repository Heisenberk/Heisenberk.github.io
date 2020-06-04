---
layout: post
title: Study a memory dump - Volatility
---

This section explains the main commands in Volatility to analyze a memory dump. 

### Introduction

Before analyzing a memory dump with Volatility, you can conduct a little background research : 

- use strings and grep to determine some simple information : 

```sh
strings mem.dmp | grep -B 10 -A 20 "some text"
```

This example prints results with 10 lines before the result and 20 lines after the result.

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

### Windows memory dump

#### Windows Environment

- See environment variables like the number of CPUs installed, the hardware architecture, the process's current directory, temporary directory, session name, computer name, user name... :

```sh
$ volatility -f mem.dmp --profile=Win7SP0x64 envars
Volatility Foundation Volatility Framework 2.4
Pid      Process              Block              Variable                       Value
-------- -------------------- ------------------ ------------------------------ -----
     296 csrss.exe            0x00000000003d1320 ComSpec                        C:\Windows\system32\cmd.exe
     296 csrss.exe            0x00000000003d1320 FP_NO_HOST_CHECK               NO
     296 csrss.exe            0x00000000003d1320 NUMBER_OF_PROCESSORS           1
     296 csrss.exe            0x00000000003d1320 OS                             Windows_NT
     296 csrss.exe            0x00000000003d1320 Path                           C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\
     296 csrss.exe            0x00000000003d1320 PATHEXT                        .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
     296 csrss.exe            0x00000000003d1320 PROCESSOR_ARCHITECTURE         AMD64
     296 csrss.exe            0x00000000003d1320 PROCESSOR_IDENTIFIER           Intel64 Family 6 Model 2 Stepping 3, GenuineIntel
     296 csrss.exe            0x00000000003d1320 PROCESSOR_LEVEL                6
     296 csrss.exe            0x00000000003d1320 PROCESSOR_REVISION             0203
     296 csrss.exe            0x00000000003d1320 PSModulePath                   C:\Windows\system32\WindowsPowerShell\v1.0\Modules\
     296 csrss.exe            0x00000000003d1320 SystemDrive                    C:
     296 csrss.exe            0x00000000003d1320 SystemRoot                     C:\Windows
     296 csrss.exe            0x00000000003d1320 TEMP                           C:\Windows\TEMP
     296 csrss.exe            0x00000000003d1320 TMP                            C:\Windows\TEMP
     296 csrss.exe            0x00000000003d1320 USERNAME                       SYSTEM
     296 csrss.exe            0x00000000003d1320 windir                         C:\Windows
```

- Get domain credentials :  

```sh
$ volatility -f mem.dmp --profile=Win7SP1x86 hashdump > password.txt
$ cat password.txt
Administrator:500:08f3a52bdd35f179c81667e9d738c5d9:ed88cccbc08d1c18bcded317112555f4::: 
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0::: 
HelpAssistant:1000:ddd4c9c883a8ecb2078f88d729ba2e67:e78d693bc40f92a534197dc1d3a6d34f::: 
SUPPORT_388945a0:1002:aad3b435b51404eeaad3b435b51404ee:8bfd47482583168a0ae5ab020e1186a9::: 
phoenix:1003:07b8418e83fad948aad3b435b51404ee:53905140b80b6d8cbe1ab5953f7c1c51::: 
ASPNET:1004:2b5f618079400df84f9346ce3e830467:aef73a8bb65a0f01d9470fadc55a411c::: 
S----:1006:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0::: 
```
Then, with <a href="https://crackstation.net/" target="_blank">crackstation</a>, or John with a good dictionary, you can recover passwords.

#### Windows Processes

- See the tree of processes :

```sh
$ volatility -f mem.dmp --profile=Win7SP0x64 pstree
Volatility Foundation Volatility Framework 2.4
Name                                                  Pid   PPid   Thds   Hnds Time                
-------------------------------------------------- ------ ------ ------ ------ --------------------
 0xfffffa80004b09e0:System                              4      0     78    489 2012-02-22 19:58:20 
. 0xfffffa8000ce97f0:smss.exe                         208      4      2     29 2012-02-22 19:58:20 
 0xfffffa8000c006c0:csrss.exe                         296    288      9    385 2012-02-22 19:58:24 
 0xfffffa8000c92300:wininit.exe                       332    288      3     74 2012-02-22 19:58:30 
. 0xfffffa8000c5eb30:services.exe                     428    332      6    193 2012-02-22 19:58:32 
.. 0xfffffa8000aa0b30:SearchIndexer.                 1800    428     12    757 2012-02-22 20:02:26 
.. 0xfffffa80007d09e0:svchost.exe                     916    428     19    443 2012-02-22 19:58:43 
.. 0xfffffa8000a4f630:svchost.exe                    1432    428     12    350 2012-02-22 20:04:14 
.. 0xfffffa800094d960:wlms.exe                       1264    428      4     43 2012-02-22 20:02:11 
.. 0xfffffa8001325950:sppsvc.exe                      816    428      5    154 2012-02-22 19:58:41 
.. 0xfffffa8000e86690:spoolsv.exe                    1076    428     12    271 2012-02-22 20:02:10 
.. 0xfffffa8001296b30:svchost.exe                     568    428     10    352 2012-02-22 19:58:34 
... 0xfffffa8000a03b30:rundll32.exe                  2016    568      3     67 2012-02-22 20:03:16
...
```

With this command, you can see suspicious processes. For example, when you replace l with I (lass.exe by Iass.exe), or when you find a suspicious cmd.exe, son of basic process (like iexplore.exe).

- See the absolute path of the executable and his associate DLLs :

```sh
$ volatility -f mem.dmp --profile=Win7SP0x64 dlllist -p 1892
Volatility Foundation Volatility Framework 2.4
************************************************************************
iexplore.exe pid:   1892
Command line : "C:\Program Files (x86)\Internet Explorer\iexplore.exe" 
Note: use ldrmodules for listing DLLs in Wow64 processes

Base                             Size          LoadCount Path
------------------ ------------------ ------------------ ----
0x0000000000080000            0xa6000             0xffff C:\Program Files (x86)\Internet Explorer\iexplore.exe
0x0000000076d40000           0x1ab000             0xffff C:\Windows\SYSTEM32\ntdll.dll
0x00000000748d0000            0x3f000                0x3 C:\Windows\SYSTEM32\wow64.dll
0x0000000074870000            0x5c000                0x1 C:\Windows\SYSTEM32\wow64win.dll
0x0000000074940000             0x8000                0x1 C:\Windows\SYSTEM32\wow64cpu.dll
```

You can delete the option ```-p/--pid``` to discover all executables and their DLLs
With this command, you can see suspicious processes. For example, if you find an executable found out of ```C:\Program Files (x86)\```, or DLL out of ```C:\Windows\SYSTEM32\```

- Print current commands : 

```sh
$ volatility -f mem.dmp --profile Win7SP1x86 cmdline

AvastUI.exe pid:   2720
Command line : "C:\Program Files\AVAST Software\Avast\AvastUI.exe" /nogui
************************************************************************
StikyNot.exe pid:   2744
Command line : "C:\Windows\System32\StikyNot.exe" 
************************************************************************
iexplore.exe pid:   2772
Command line : "C:\Users\John Doe\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\iexplore.exe" 
************************************************************************
SearchIndexer. pid:   2900
Command line : C:\Windows\system32\SearchIndexer.exe /Embedding
************************************************************************
wmpnetwk.exe pid:   3176
Command line : "C:\Program Files\Windows Media Player\wmpnetwk.exe"
************************************************************************
```

- Print registry values : 

```sh
$ volatility -f mem.dmp --profile Win7SP1x86 printkey -K "Software\Microsoft\Windows\CurrentVersion\Run"

----------------------------
Registry: \??\C:\Users\John Doe\ntuser.dat
Key name: Run (S)
Last updated: 2013-01-12 14:13:19

Subkeys:

Values:
REG_SZ        RESTART_STICKY_NOTES : (S) C:\Windows\System32\Malware.exe
REG_SZ        IEPreload       : (S) "C:\Users\John Doe\AppData\Roaming\malware.exe
```

- Dump a process's executable : 

```sh
$ volatility -f mem.dmp --profile=Win7SP0x64 procdump -D dmp/ -p 296
Volatility Foundation Volatility Framework 2.4
************************************************************************
Dumping csrss.exe, pid:    296 output: executable.296.exe

$ file dmp/executable.296.exe 
dump/executable.296.exe: PE32+ executable for MS Windows (native) Mono/.Net assembly
```

- Extract all memory resident pages in a process into an individual file : 

```sh
$ volatility -f mem.dmp --profile=Win7SP0x64 memdump -p 4 -D dmp/
Volatility Foundation Volatility Framework 2.4
************************************************************************
Writing System [     4] to 4.dmp

$ ls -alh dmp/4.dmp 
-rw-r--r--  1 Michael  staff   111M Jun 24 15:47 dump/4.dmp
```

#### Windows files

- List cached files : 

```sh
$ volatility -f mem.dmp --profile=Win7SP0x64 filescan
Volatility Foundation Volatility Framework 2.4
Offset(P)            #Ptr   #Hnd Access Name
------------------ ------ ------ ------ ----
0x000000000126f3a0     14      0 R--r-d \Windows\System32\mswsock.dll
0x000000000126fdc0     11      0 R--r-d \Windows\System32\ssdpsrv.dll
0x000000000468f7e0      6      0 R--r-d \Windows\System32\cryptsp.dll
0x000000000468fdc0     16      0 R--r-d \Windows\System32\Apphlpdm.dll
0x00000000048223a0      1      1 ------ \Endpoint
0x0000000004822a30     16      0 R--r-d \Windows\System32\kerberos.dll
0x0000000004906070     13      0 R--r-d \Windows\System32\wbem\repdrvfs.dll
0x0000000004906580      9      0 R--r-d \Windows\SysWOW64\netprofm.dll
0x0000000004906bf0      9      0 R--r-d \Windows\System32\wbem\wmiutils.dll
0x00000000049ce8e0      2      1 R--rwd \$Extend\$ObjId
0x00000000049cedd0      1      1 R--r-d \Windows\System32\en-US\vsstrace.dll.mui
0x0000000004a71070     17      1 R--r-d \Windows\System32\en-US\pnidui.dll.mui
0x0000000004a71440     11      0 R--r-d \Windows\System32\nci.dll
0x0000000004a719c0      1      1 ------ \srvsvc
```

- extract mapped memory and cached files of a process : 

```sh
$ volatility -f mem.dmp --profile=Win7SP0x86 dumpfiles -n -p 3224 -D dmp/
```

- extract all cached files (very expensive) : 

```sh
$ volatility -f mem.dmp --profile=Win7SP0x86 dumpfiles -D dmp/
```

- extract a file at a specific offset (unsafe mode), using the file name (```-n```) and summarizing informations in a specific text file (```summary.txt```) :

```sh
$ volatility -f mem.dmp --profile=Win7SP0x86 dumpfiles -Q 0x00000000007f3270 -D dmp/ -u -n -S summary.txt
```

- extract files with a specific substring in the name : 

```sh
$ volatility -f mem.dmp --profile=Win7SP0x86 dumpfiles -D output/ -r ABCDE -i -S summary.txt
```

#### Windows networking

- View active TCP connections : 

```sh
$ volatility -f mem.dmp --profile=Win2003SP2x64 connections
Volatile Systems Volatility Framework 2.1_alpha
Offset(V)          Local Address             Remote Address               Pid
------------------ ------------------------- ------------------------- ------
0xfffffadfe6f2e2f0 172.16.237.150:1408       72.246.25.25:80             2136
0xfffffadfe72e8080 172.16.237.150:1369       64.4.11.30:80               2136
0xfffffadfe622d010 172.16.237.150:1403       74.125.229.188:80           2136
0xfffffadfe62e09e0 172.16.237.150:1352       64.4.11.20:80               2136
0xfffffadfe6f2e630 172.16.237.150:1389       209.191.122.70:80           2136
0xfffffadfe5e7a610 172.16.237.150:1419       74.125.229.187:80           2136
0xfffffadfe7321bc0 172.16.237.150:1418       74.125.229.188:80           2136
0xfffffadfe5ea3c90 172.16.237.150:1393       216.115.98.241:80           2136
0xfffffadfe72a3a80 172.16.237.150:1391       209.191.122.70:80           2136
0xfffffadfe5ed8560 172.16.237.150:1402       74.125.229.188:80           2136
```

This command works for x86 and x64 Windows XP and Windows 2003 Server only. 

- Detect listening sockets for any protocol (TCP, UDP, RAW, etc) : 

```sh
$ volatility -f mem.dmp --profile=Win2003SP2x64 sockets
Volatility Foundation Volatility Framework 2.4
Offset(V)             PID   Port  Proto Protocol        Address         Create Time
------------------ ------ ------ ------ --------------- --------------- -----------
0xfffffadfe71bbda0    432   1025      6 TCP             0.0.0.0         2012-01-23 18:20:01 
0xfffffadfe7350490    776   1028     17 UDP             0.0.0.0         2012-01-23 18:21:44 
0xfffffadfe6281120    804    123     17 UDP             127.0.0.1       2012-06-25 12:40:55 
0xfffffadfe7549010    432    500     17 UDP             0.0.0.0         2012-01-23 18:20:09 
0xfffffadfe5ee8400      4      0     47 GRE             0.0.0.0         2012-02-24 18:09:07 
0xfffffadfe606dc90      4    445      6 TCP             0.0.0.0         2012-01-23 18:19:38 
0xfffffadfe6eef770      4    445     17 UDP             0.0.0.0         2012-01-23 18:19:38 
0xfffffadfe7055210   2136   1321     17 UDP             127.0.0.1       2012-05-09 02:09:59 
0xfffffadfe750c010      4    139      6 TCP             172.16.237.150  2012-06-25 12:40:55 
0xfffffadfe745f610      4    138     17 UDP             172.16.237.150  2012-06-25 12:40:55 
0xfffffadfe6096560      4    137     17 UDP             172.16.237.150  2012-06-25 12:40:55 
0xfffffadfe7236da0    720    135      6 TCP             0.0.0.0         2012-01-23 18:19:51 
0xfffffadfe755c5b0   2136   1419      6 TCP             0.0.0.0         2012-06-25 12:42:37 
0xfffffadfe6f36510   2136   1418      6 TCP             0.0.0.0         2012-06-25 12:42:37 
```

This command works for x86 and x64 Windows XP and Windows 2003 Server only. 

- Scan for network artifacts in 32 and 64-bit Windows Vista, Windows 2008 Server and Windows 7 memory dumps. This finds TCP endpoints, TCP listeners, UDP endpoints, and UDP listeners : 

```sh
$ volatility -f mem.dmp --profile=Win7SP0x64 netscan
Volatility Foundation Volatility Framework 2.4
Offset(P)  Proto    Local Address                  Foreign Address      State            Pid      Owner          Created
0xf882a30  TCPv4    0.0.0.0:135                    0.0.0.0:0            LISTENING        628      svchost.exe    
0xfc13770  TCPv4    0.0.0.0:49154                  0.0.0.0:0            LISTENING        916      svchost.exe    
0xfdda1e0  TCPv4    0.0.0.0:49154                  0.0.0.0:0            LISTENING        916      svchost.exe    
0xfdda1e0  TCPv6    :::49154                       :::0                 LISTENING        916      svchost.exe    
0x1121b7b0 TCPv4    0.0.0.0:135                    0.0.0.0:0            LISTENING        628      svchost.exe    
0x1121b7b0 TCPv6    :::135                         :::0                 LISTENING        628      svchost.exe    
0x11431360 TCPv4    0.0.0.0:49152                  0.0.0.0:0            LISTENING        332      wininit.exe    
0x11431360 TCPv6    :::49152                       :::0                 LISTENING        332      wininit.exe    
[snip]
0x17de8980 TCPv6    :::49153                       :::0                 LISTENING        444      lsass.exe      
0x17f35240 TCPv4    0.0.0.0:49155                  0.0.0.0:0            LISTENING        880      svchost.exe    
0x17f362b0 TCPv4    0.0.0.0:49155                  0.0.0.0:0            LISTENING        880      svchost.exe    
0x17f362b0 TCPv6    :::49155                       :::0                 LISTENING        880      svchost.exe    
0xfd96570  TCPv4    -:0                            232.9.125.0:0        CLOSED           1        ?C?            
0x17236010 TCPv4    -:49227                        184.26.31.55:80      CLOSED           2820     iexplore.exe   
0x1725d010 TCPv4    -:49359                        93.184.220.20:80     CLOSED           2820     iexplore.exe   
0x17270530 TCPv4    10.0.2.15:49363                173.194.35.38:80     ESTABLISHED      2820     iexplore.exe   
0x17285010 TCPv4    -:49341                        82.165.218.111:80    CLOSED           2820     iexplore.exe   
0x17288a90 TCPv4    10.0.2.15:49254                74.125.31.157:80     CLOSE_WAIT       2820     iexplore.exe   
0x1728f6b0 TCPv4    10.0.2.15:49171                204.245.34.130:80    ESTABLISHED      2820     iexplore.exe   
0x17291ba0 TCPv4    10.0.2.15:49347                173.194.35.36:80     CLOSE_WAIT       2820     iexplore.exe   
[snip]
0x17854010 TCPv4    -:49168                        157.55.15.32:80      CLOSED           2820     iexplore.exe   
0x178a2a20 TCPv4    -:0                            88.183.123.0:0       CLOSED           504      svchost.exe    
0x178f5b00 TCPv4    10.0.2.15:49362                173.194.35.38:80     CLOSE_WAIT       2820     iexplore.exe   
0x17922910 TCPv4    -:49262                        184.26.31.55:80      CLOSED           2820     iexplore.exe   
0x17a9d860 TCPv4    10.0.2.15:49221                204.245.34.130:80    ESTABLISHED      2820     iexplore.exe   
0x17ac84d0 TCPv4    10.0.2.15:49241                74.125.31.157:80     CLOSE_WAIT       2820     iexplore.exe   
0x17b9acf0 TCPv4    10.0.2.15:49319                74.125.127.148:80    CLOSE_WAIT       2820     iexplore.exe   
0x10f38d70 UDPv4    10.0.2.15:1900                 *:*                                   1736     svchost.exe    2012-02-22 20:04:12 
0x173b3dc0 UDPv4    0.0.0.0:59362                  *:*                                   1736     svchost.exe    2012-02-22 20:02:27 
0x173b3dc0 UDPv6    :::59362                       *:*                                   1736     svchost.exe    2012-02-22 20:02:27 
0x173b4cf0 UDPv4    0.0.0.0:3702                   *:*                                   1736     svchost.exe    2012-02-22 20:02:27 
0x173b4cf0 UDPv6    :::3702                        *:*   
```

For more information, check this **<a href="https://github.com/volatilityfoundation/volatility/wiki/Command-Reference" target="_blank">link</a>**
