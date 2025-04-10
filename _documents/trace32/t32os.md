---
permalink: /documents/trace32/t32os/
title: Trace32
excerpt: "Trace32 OS awareness"
comments: false
toc: true
---


## Trace32 OS awareness

Trace32’s OS awareness means it can recognize all tasks and flows of the operating system.<br>
The process broadly involves <B>MMU setup</B>, <B>TASK config</B>, and <B>MENU reprograming</B>.

## Trace32 Linux Awareness

To implement Linux awareness,<br>
proceed sequentially with MMU setup, Task config, and Menu reprogramming.<br>

### MMU setup

<p align="center">
  <img src="/documents/images/trace32/t32os0.png" alt="t32os0" width="640" height="480"><br>
</p>
<br>
In <span style="{{ site.code }}">B::SYStem</span> , check MMUSPACES or use the command SYStem.Option.MMUSPACES ON<br>
to enable the MMUSPACES option.<br>
<br>

<p align="center">
  <img src="/documents/images/trace32/t32os1.png" alt="t32os1" width="640" height="480"><br>
</p>
<br>
Use <span style="{{ site.code }}">B::y.list</span> to find the text symbol, which is the starting point of the code.<br>
The starting virtual address is <span style="{{ site.code }}">0xFFFF8000800000000</span> .<br>
<br>

<p align="center">
  <img src="/documents/images/trace32/t32os2.png" alt="t32os2" width="640" height="480"><br>
</p>
<br>
Find the end address of the virtual address, which is <span style="{{ site.code }}">0xFFFF800082030000</span> .<br>
<br>

<p align="center">
  <img src="/documents/images/trace32/t32os3.png" alt="t32os3" width="640" height="480"><br>
</p>
<br>
Use the <span style="{{ site.code }}">B::MMU.List.PageTable</span> or <span style="{{ site.code }}">B::MMU.List.EL1PageTable</span> command to check the MMU VA-to-PA mapping table.<br>
You can also see why there’s a slight difference from the text virtual address <B>(0xFFFF8000800000000)</B>.<br>
This is because it’s not the exact starting address.<br>
<br>

<p align="center">
  <img src="/documents/images/trace32/t32os4.png" alt="t32os4" width="640" height="480"><br>
</p>
<br>
Use <span style="{{ site.code }}">B::sYmbol.List.Section</span> to check the <span style="{{ site.code }}">.head.text</span> address.<br>
Now you can confirm the exact starting address.<br>
Match the VA starting address with the PA starting address to set up OS awareness.<br>
<br>

<p align="center">
  <img src="/documents/images/trace32/t32os5.png" alt="t32os5" width="640" height="480"><br>
</p>
<br>
When checking the MMU table again, the starting physical address is <span style="{{ site.code }}">AN:0x80000000</span> .<br>
The <span style="{{ site.code }}">.head.text symbol</span> is mapped to the physical address <span style="{{ site.code }}">AN:0x80000000</span> .<br>
The AN class indicates that <B>A</B> stands for physical address and <B>N</B> stands for <B>EL1</B>.<br>
Execute the following command for Linux awareness.<br>
<br>

<p align="center">
  <img src="/documents/images/trace32/t32os6.png" alt="t32os6" width="640" height="480"><br>
</p>
<br>
To set up MMU translation, find the swapper_pg_dir symbol.<br>
<br>
Use the command:<br>
<span style="{{ site.code }}">B::MMU.FORMAT LINUXSWAP3 {swapper symbol}</span><br>
<br>
Here, input it as follows:
```
B::MMU.FORMAT LINUXSWAP3 \\vmlinux\Global\swapper_pg_dir
```
<br>

For translation,<br>
input the symbol’s starting virtual address, ending virtual address, and starting physical address:<br>
<span style="{{ site.code }}">B::TRANSlation.Create {symbol start virtual address}—{symbol end virtual address - 1} {symbol start physical address}</span><br>
<br>
Here, input it as follows:
```
B::TRANSlation.Create 0xFFFF8000000000000—0xFFFF80008202FFFF a:0x80000000
```
<br>

Since manually entering these addresses can be difficult,<br>
referencing symbols works just as well:
```
B::TRANSlation.Create (\\vmlinux\Global\_text-0x80000000)--(\\vmlinux\Global\_end-1) a:0x80000000
```
<br>

Next, set the entire virtual address range:<br>
<span style="{{ site.code }}">B::TRANSlation.COMMON {virtual address system start address}—{virtual address system end address}</span><br>
<br>
Here, input it as follows:
```
B::TRANSlation.COMMON 0xFFFF000000000000—0xFFFFFFFFFFFFFFFF
```
<br>

Once the setup is complete, enable translation:
```
B::TRANSlation.TableWalk ON
B::TRANSlation.ON
```
<br>

<B>The commands can be summarized as follows:</B>
```
B::MMU.FORMAT LINUXSWAP3 \\vmlinux\Global\swapper_pg_dir
B::TRANSlation.Create (\\vmlinux\Global\_text-0x80000000)--(\\vmlinux\Global\_end-1) a:0x80000000
B::TRANSlation.COMMON 0xFFFF000000000000—0xFFFFFFFFFFFFFFFF
B::TRANSlation.TableWalk ON
B::TRANSlation.ON
```
<br>

With this, the MMU setup is complete.<br>
While MMU setup is complex, TASK setup and MENU addition are not as difficult.<br>
<br>

### Task config
<p align="center">
  <img src="/documents/images/trace32/t32os7.png" alt="t32os7" width="640" height="480"><br>
</p>
<p align="center">
  <img src="/documents/images/trace32/t32os8.png" alt="t32os8" width="640" height="480"><br>
</p>
<br>
Using <span style="{{ site.code }}">B::TASK.CONFIG {task file}</span> to proceed.<br>
Just like when loading an ELF file,<br>
you can either directly specify the file path or use \* to select the file manually.<br>
For reference, in the image, the file was written with a relative path.<br>
Using <span style="{{ site.code }}">B::cd</span> will display the current path where the program is running.<br>
You can confirm that a new ‘linux’ toolbar has been created.<br>
<br>

<p align="center">
  <img src="/documents/images/trace32/t32os9.png" alt="t32os9" width="640" height="480"><br>
</p>
<br>
Using <span style="{{ site.code }}">B::MENU.REPROGRAM {menu file}</span> to proceed.<br>
Just like when loading an ELF file,<br>
you can either directly specify the file path or use \* to select the file manually.<br>
For reference, in the image, the file was written with a relative path.<br>
Using <span style="{{ site.code }}">B::cd</span> will display the current path where the program is running.<br>
<br>

### summary

To summarize all the commands for Linux awareness,
```
B::MMU.FORMAT LINUXSWAP3 \\vmlinux\Global\swapper_pg_dir
B::TRANSlation.create (\\vmlinux\Global\_text-0x80000000)--(\\vmlinux\Global\_end-1) a:0x80000000
B::TRANSlation.COMMON 0xFFFF000000000000—0xFFFFFFFFFFFFFFFF
B::TRANSlation.TableWalk ON
B::TRANSlation.ON
B::TASK.CONFIG ~~/demo/arm/kernel/linux/awareness/linux.t32
B::menu.ReProgram ~~/demo/arm/kernel/linux/awareness/linux.men
```
<br>

Now, through the ‘linux’ menu, you can use various powerful awareness features.<br>
You can debug on a task-by-task basis or verify things in the debugging program<br>
that can also be checked on the system.<br>

<p align="center">
  <img src="/documents/images/trace32/t32os10.png" alt="t32os10" width="640" height="480"><br>
</p>
<br>
Like this, you can check the currently running processes,<br>
<br>

<p align="center">
  <img src="/documents/images/trace32/t32os11.png" alt="t32os11" width="640" height="480"><br>
</p>
<br>
Like this, you can also check kernel messages.<br>
<br>



## Trace32 Xen Hypervisor Awareness
