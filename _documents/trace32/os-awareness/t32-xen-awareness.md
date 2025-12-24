---
permalink: /documents/trace32/t32-xen-awareness/
title: Trace32 xen awareness
excerpt: "Trace32 xen awareness"
comments: false
toc: true
---


## Trace32 Xen Awareness

To implement Xen awareness,<br>
proceed sequentially with MMU setup, Task config, and Menu reprogramming.<br>
<br>

```
[Warning]

This document is for those who have some understanding of trace32 and osawareness.
If you are not familiar with the awareness setting,
please refer to the linux awareness document first.
```
Go to [Linux Awareness](/documents/trace32/os_awareness/t32linux)
<br>

Hypervisor awareness settings may vary slightly depending on the system design.<br>
Examples of systems in this document are as follows.
```

 +------- VM1 -------+ +------- VM2 -------+
 |      (dom0)       | |      (domu1)      |
 | Linux Kernel 5.10 | | Linux Kernel 5.10 |
 +-------------------+ +-------------------+
 ===========================================
 +--------------- HYPERVISOR --------------+
 |             Xen Kernel 4.16             |
 +-----------------------------------------+

```
<br>

xen also needs to turn off watchdog before connecting the debugger.<br>
However, in the case of xen,<br>
the daemon performs the wdt timer operation in the background.
```
$ systemctl stop xen-watchdog.service
```
<br>

### MMU setup

<img src="/assets/images/documents/trace32/t32xen0.png" alt="t32xen0" width="640" height="480"><br>
In <span style="{{ site.code }}">B::SYStem</span> , check MMUSPACES or use the command <span style="{{ site.code }}">B::SYStem.Option.MMUSPACES ON</span><br>
to enable the MMUSPACES option.<br>
and check MACHINESPACES or use the command <span style="{{ site.code }}">B::SYStem.Option.MACHINESPACES ON</span><br>
to enable the MMUSPACES option.<br>

In an environment where xen and other hypervisors are set up,<br>
several symbols are raised together, so machine space is used to make it easier to distinguish.<br>
Of course, machine space is not a mendatory.<br>

The use of machine space causes some changes in the address system used by the debugger program.
```
address => {class}:{machine space}:::{address}
```
<br>

With macinespace enabled, the command when uploading the xen kernel symbol is as follows.
```
Data.LOAD.Elf &xen_symbol H:0:::0 /noclear /nocode /anysym /macro /gnu /Name xen
sYmbol.sourcePATH.Translate "&xen_invalid_path" "&xen_correct_path"
```
<br>

Now we need to prepare mmutable matching for awareness.<br>
However, since the hypervisor will use the entire area anyway, I set it up as follows.
```
MMU.FORMAT STD
TRANSlation.COMMON H:0:::0x0--0xffffffffffffffff
TRANSlation.TableWalk ON
TRANSlation.ON
```
<br>

And you can add the config file.<br>
The <span style="{{ site.code }}">Task.config</span> command is replaced by the <span style="{{ site.code }}">EXTENSION.LOAD</span> command<br>
because it cannot carry information about machine space. (based on <B>Power View 2023</B>)
```
EXTension.LOAD &xen_cfg_file /Machine 0 /NAME xen /ACCESS H:
MENU.ReProgram &xen_men_file
```
<br>

Remember the system design of this document?<br>
I will put up the <span style="{{ site.code }}">dom0</span> symbol first.
```
Data.LOAD.Elf &dom0_symbol N:1:::0 /noclear /nocode /anysym /macro /gnu /Name dom0
sYmbol.sourcePATH.Translate "&dom0_invalid_path" "&dom0_correct_path"
```
<br>

You need to find an address for table mapping.<br>
The hypervisor performs a <span style="{{ site.code }}">two-stage conversion</span> .<br>
The physical address converted through mmu in the VM is actually not a real physical address.<br>
<br>

To explain it through design,
```

+----+                 +-----+                 +-----+
| VM | <---> (VA) <--- | MMU | ---> (IPA) <--- | HYP | ---> (PA)
+----+                 +-----+                 +-----+

```
<br>

The address translated to the MMU is the <B>I</B>mmediate <B>P</B>hysical <B>A</B>ddress.<br>
This address is actually the address where the hypervisor virtualized the real physical address once.<br>
In other words, from the point of view of VM, the address system is mistaking IPA for real PA!<br>
<br><br>
The Trace32 debugger uses ' <span style="{{ site.code }}">I</span> ' as a class representing the IPA address system.<br>

<br>
The process for <span style="{{ site.code }}">dom0</span> mmu setup is as follows.<br>
<br>

<img src="/assets/images/documents/trace32/t32xen1.png" alt="t32xen1" width="640" height="480"><br>
Use <span style="{{ site.code }}">B::y.list</span> to find the text symbol, which is the starting point of the code.<br>
The starting virtual address is <span style="{{ site.code }}">NP:1:0xFFFFFFC010000000</span> .<br>
<br>

<img src="/assets/images/documents/trace32/t32xen2.png" alt="t32xen2" width="640" height="480"><br>
Find the end address of the virtual address, which is <span style="{{ site.code }}"> NP:1:0xFFFFFFC0110E0000</span> .<br>
<br>

<img src="/assets/images/documents/trace32/t32xen3.png" alt="t32xen3" width="640" height="480"><br>
Use the <span style="{{ site.code }}">B::MMU.List.EL1PageTable /MACHINE 1</span> command to check the MMU VA-to-IPA mapping table.<br>
When machine space is allocated in this way, it can be distinguished by the MACHINE option.<br>
You can also see why there’s a slight difference from the text virtual address <B>(I:1:::0x60000000)</B>.<br>
<br>

Since <span style="{{ site.code }}">dom0</span> is a Linux kernel,<br>
it constructs MMU translation tables in the same way that [linux awareness](/documents/trace32/os_awareness/t32linux/) did.<br>
<br>
B::MMU.FORMAT LINUXSWAP3 \\dom0\Global\swapper_pg_dir<br>
B::TRANSlation.Create N:1:::0xFFFFFFC0100000000—0xFFFFFFC0110E00000 I:1:0x60000000<br>

And set the entire range of virtual addresses.<br>
B::TRANSlation.COMMON N:1:::0xFFFFFF0000000000--0xFFFFFFFFFFFFFFFF<br>

Turn on Translation when you are done setting up.<br>
B::TRANSlation.TableWalk ON<br>
B::TRANSlation.ON<br>

You can also refer to the symbol directly<br>
because it is cumbersome to write the address directly.<br>
So, the command is summarized as follows.
```
B::MMU.FORMAT LINUXSWAP3 \\vmlinux\Global\swapper_pg_dir
B::TRANSlation.create (\\dom0\Global\_text)--(\\dom0\Global\_end-1) I:1:::0x60000000
B::TRANSlation.COMMON 0xFFFF000000000000—0xFFFFFFFFFFFFFFFF
B::TRANSlation.TableWalk ON
B::TRANSlation.ON
B::EXTension.LOAD &dom0_cfg_file /Machine 1 /NAME dom0
```
I also added the linux kernel config file, and the menu config is set to xen menu, so it is omitted.<br>
<br>

<span style="{{ site.code }}">domu1</span> goes through the same process. machine space uses space 2.
```
Data.LOAD.Elf &domu1_symbol N:2:::0 /noclear /nocode /anysym /macro /gnu /Name domu1
sYmbol.sourcePATH.Translate "&domu1_invalid_path" "&domu1_correct_path"
```
<br>

<img src="/assets/images/documents/trace32/t32xen4.png" alt="t32xen4" width="640" height="480"><br>
Use <span style="{{ site.code }}">B::y.list</span> to find the text symbol, which is the starting point of the code.<br>
The starting virtual address is <span style="{{ site.code }}">NP:2:0xFFFFFFC010000000</span> .<br>
<br>

<img src="/assets/images/documents/trace32/t32xen5.png" alt="t32xen5" width="640" height="480"><br>
Find the end address of the virtual address, which is <span style="{{ site.code }}"> NP:2:0xFFFFFFC0110E0000</span> .<br>
<br>

<img src="/assets/images/documents/trace32/t32xen6.png" alt="t32xen6" width="640" height="480"><br>
Use the <span style="{{ site.code }}">B::MMU.List.EL1PageTable /MACHINE 2</span> command to check the MMU VA-to-IPA mapping table.<br>
When machine space is allocated in this way, it can be distinguished by the MACHINE option.<br>
You can also see why there’s a slight difference from the text virtual address <B>(I:2:::0x40000000)</B>.<br>
<br>

Since <span style="{{ site.code }}">domu1</span> is a Linux kernel,<br>
it constructs MMU translation tables in the same way that [linux awareness](/documents/trace32/os_awareness/t32linux/) did.<br>
<br>

B::MMU.FORMAT LINUXSWAP3 \\domu1\Global\swapper_pg_dir<br>
B::TRANSlation.Create N:2:::0xFFFFFFC0100000000—0xFFFFFFC0110E00000 I:2:0x40000000<br>

And set the entire range of virtual addresses.<br>
B::TRANSlation.COMMON N:2:::0xFFFFFF0000000000--0xFFFFFFFFFFFFFFFF<br>

Turn on Translation when you are done setting up.<br>
B::TRANSlation.TableWalk ON<br>
B::TRANSlation.ON<br>

You can also refer to the symbol directly<br>
because it is cumbersome to write the address directly.<br>
So, the command is summarized as follows.
```
B::MMU.FORMAT LINUXSWAP3 \\vmlinux\Global\swapper_pg_dir
B::TRANSlation.create (\\domu1\Global\_text)--(\\domu1\Global\_end-1) I:2:::0x40000000
B::TRANSlation.COMMON 0xFFFF000000000000—0xFFFFFFFFFFFFFFFF
B::TRANSlation.TableWalk ON
B::TRANSlation.ON
B::EXTension.LOAD &domu1_cfg_file /Machine 2 /NAME domu1
```
I also added the linux kernel config file, and the menu config is set to xen menu, so it is omitted.<br>

### Summary

The cmm script is summarized as follows.
```
; initialize
SYStem.Option.MMUSPACES ON
SYStem.Option.MACHINESPACE ON
sYmbol.RESet

; xen setup
Data.LOAD.Elf &xen_symbol H:0:::0 /noclear /nocode /anysym /macro /gnu /Name xen
sYmbol.sourcePATH.Translate "&xen_invalid_path" "&xen_correct_path"
MMU.FORMAT STD
TRANSlation.COMMON H:0:::0x0--0xffffffffffffffff
TRANSlation.TableWalk ON
TRANSlation.ON
EXTension.LOAD &xen_cfg_file /Machine 0 /NAME xen /ACCESS H:
MENU.ReProgram &xen_men_file

; dom0 setup
Data.LOAD.Elf &dom0_symbol N:1:::0 /noclear /nocode /anysym /macro /gnu /Name dom0
sYmbol.sourcePATH.Translate "&dom0_invalid_path" "&dom0_correct_path"

B::MMU.FORMAT LINUXSWAP3 \\vmlinux\Global\swapper_pg_dir
B::TRANSlation.create (\\dom0\Global\_text)--(\\dom0\Global\_end-1) I:1:::0x60000000
B::TRANSlation.COMMON 0xFFFF000000000000—0xFFFFFFFFFFFFFFFF
B::TRANSlation.TableWalk ON
B::TRANSlation.ON
B::EXTension.LOAD &dom0_cfg_file /Machine 1 /NAME dom0

; domu1 setup
Data.LOAD.Elf &domu1_symbol N:2:::0 /noclear /nocode /anysym /macro /gnu /Name domu1
sYmbol.sourcePATH.Translate "&domu1_invalid_path" "&domu1_correct_path"
B::MMU.FORMAT LINUXSWAP3 \\vmlinux\Global\swapper_pg_dir
B::TRANSlation.create (\\domu1\Global\_text)--(\\domu1\Global\_end-1) I:2:::0x40000000
B::TRANSlation.COMMON 0xFFFF000000000000—0xFFFFFFFFFFFFFFFF
B::TRANSlation.TableWalk ON
B::TRANSlation.ON
B::EXTension.LOAD &domu1_cfg_file /Machine 2 /NAME domu1
```
