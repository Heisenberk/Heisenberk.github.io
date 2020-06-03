---
layout: post
title: Make a memory dump to analyze it (Volatility)
---

This section explains how to make a memory dump on Windows and Linux. 

### Make a memory dump on Windows

#### Make a memory dump of a specific process

Open the task manager and determine the PID of the targeted process.

With **procdump (you can find it <a href="{{ site.baseurl }}/downloads/procdump.zip" target="_blank">here</a>)** : 

```sh
procdump.exe <pid>
```

#### Make a memory dump of all the memory

With **DumpIt (you can find it <a href="{{ site.baseurl }}/downloads/DumpIt.zip" target="_blank">here</a>)** : 

```sh
DumpIt.exe <pid>
```

Then , you can analyse the result with volatility.

