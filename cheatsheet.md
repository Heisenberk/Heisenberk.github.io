---
layout: cheatsheet
title: CheatSheet
permalink: /cheatsheet/
---

### Useful websites

|Tool|Description|
|-----|----------|
|**<a href="https://gchq.github.io/CyberChef/" target="_blank">CyberChef.com</a>**|to encode/decode Base64, hexadecimal, ciphers ... : |
|**<a href="https://crackstation.net/" target="_blank">CrackStation.net</a>**| to decode password written in different hashes|
|**<a href="http://www.decompiler.com/" target="_blank">Decompiler.com</a>** | to transform APK file to Java files representing the source code|


### Extract files 

To extract data from files : 

|Tool|Description|Usage|
|-----|-------------|-------|
|```foremost```|Extract data from specific files|```foremost -o result/ -t zip -i mem.dmp``` to extract zip files|
|```binwalk```|Extract all data|```binwalk --dd='.*' mem.dmp```|

### Bruteforce

- crunch [MIN] [MAX] [CONTENT] : 

```sh
crunch 4 10 “abcdefghijklmnopqrstuvwxyz”
```

- john [HASHFILE] --wordlist=/usr/share/wordlists/rockyou.txt

### Analyze an executable 

|Tool|Description|
|-----|----------|
|**<a href="https://www.virustotal.com" target="_blank">VirusTotal.com</a>** | to analyse the behaviour of an executable (APK, PE, ELF...)|


### Analyze a live memory dump

|Tool|Description|
|-----|----------|
|**<a href="https://github.com/volatilityfoundation/volatility/wiki/Command-Reference" target="_blank">Volatility Windows</a>** | to analyze a Windows live memory dump|
|**<a href="https://github.com/volatilityfoundation/volatility/wiki/Linux-Command-Reference" target="_blank">Volatility Linux</a>**| to analyze a Linux live memory dump|
|**<a href="https://github.com/volatilityfoundation/volatility/wiki/Mac-Command-Reference" target="_blank">Volatility MacOSX</a>**| to analyze a MacOSX live memory dump|

### Analyze a random file 

|Tool|Description|Usage|
|-----|-------------|-------|
|file|Checks the type of file|file -filename|
|exiftool|Gives the basic metadata|exiftool - filename |
|binwalk|Shows the embedded files| binwalk -filename |
|strings | Gives out all printable characters | strings -filename |
|foremost | Extracts out any embedded files | foremost -filename |
|pngcheck | Details about a png image | pngcheck –options -filename |
|ffmpeg | Check the integrity of audio files | ffmpeg –options -filename |

### Analyze an audio file 

| Tool | Description | Usage|
|------|-------------|------|
|Audacity| This is a great tool in analysing, modifying and revealing any data present inside audio, mostly used in analysing audio files| audacity -filename| 
|Sonic Visualiser| Yet another similar tool like Audacity, which also cud be used in investigating audio files| sonic-visualiser -filename|
|Deepsound| This is a tool which is used to hide/reveal any data in audio file using a password| It is a Windows apllication |

### Analyze a network packet capture

|Tool| Description | Usage|
|----|-------------|------|
|Wireshark| Wireshark is a free and open source packet analyzer. It is used for network troubleshooting, analysis, software and communications protocol development | wireshark filename.pcap|
|Tcpdump| tcpdump is a common packet analyzer that runs under the command line. It allows the user to display TCP/IP and other packets being transmitted or received over a network to which the computer is attached | tcpdump -options |
|Network Miner| NetworkMiner is a Network Forensic Analysis Tool (NFAT) for Windows. NetworkMiner can be used as a passive network sniffer/packet capturing tool in order to detect operating systems, sessions, hostnames, open ports etc. without putting any traffic on the network | GUI application|

### Analyze a disk image

| Tool | Description | Usage |
|------|-------------|-------|
|Fdisk| For computer file systems, fdisk is a command-line utility that provides disk partitioning functions | fdsik -lu filename|
|mmls | mmls displays the contents of a volume system (media management). In general, this is used to list the partition table contents so that you can determine where each partition starts. The output identifies the type of partition and its length, which makes it easy to use 'dd' to extract the partitions| mmls filename |
|TestDisk| Testdisk is powerful free data recovery software. It was primarily designed to help recover lost partitions and/or make non-booting disks bootable again when these symptoms are caused by faulty software | testdisk filename|
|Autopsy| Autopsy is computer software that makes it simpler to deploy many of the open source programs and plugins used in The Sleuth Kit. The graphical user interface displays the results from the forensic search of the underlying volume making it easier for investigators to flag pertinent sections of data| GUI Application|
|OSForensics| OSForensics is a digital computer forensic application which lets you extract and analyse digital data evidence efficiently and with ease. It discovers, identifies and manages ie uncovers everything hidden inside your computer systems and digital storage devices. | GUI Application | 

### Ascii Table 

![img ascii]({{ site.baseurl }}/images/cheatsheet/ascii-table.png)