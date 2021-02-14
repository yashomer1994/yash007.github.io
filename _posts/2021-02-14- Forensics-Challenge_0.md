---
layout: post
title:  "Forensic Basics Challenge_0"
date:   2021-02-14
categories: C C++ Asssembly Python 
title: Forensic Bacis Challenge_0
---

I am very curious about memory forensics, so i decided to start with Introdution of Memory Forensics using CTF Challenge Samples.

---
[](#header-1)**Definition**
---

Memory forensic refers to the analysis of the volatile memory dump of Virtual Image or a Physical System Image. This analysis is carried out by security researchers to investigate and identify malicious behaviour or attacks which got missed by UserLand Softwares.

---
[](#header-1)**Memory Dump**
---

A Snapshot of A Virtual Machine image or a physical memory dump which may contain about the valuable forensics data about the system behaviour and root cause of a crash.

---
[](#header-1)**Analysis-Stage_1**
---

Our first stage will be to analyse the Memory dump file using tool Volatility Framework.

We will thing we will use the **PROFILE** to determine the Image specifics.

The **Profile** Tells about the OS of the image from which machine the dump data was extracted.

![](https://yashomer1994.github.io/yash007.github.io/assets/forensics/challenge0/imageinfo.png)

Volatility framework provides the details about of image, which profile to be used.

As a Security Researcher, Following specifications are analysed before going any further :

1. Currently Running Processes.
2. Commands executed.
3. Processes which have been terminated.
4. Browser History.

So , to list the current running processes, we will use the following command as shown in image below.

![](https://yashomer1994.github.io/yash007.github.io/assets/forensics/challenge0/process.png)

We can see the list of processes which were running when the memory dump was carried out. 



