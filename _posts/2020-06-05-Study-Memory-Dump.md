---
layout: post
title: Study a live memory dump
---

This section explains how to analyze a memory dump before using Volatility. 

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

Now you can study the memory dump with Volatility : firstly, find the correct profile and then, use different commands to understand the content of the memory dump.
