---
permalink: /documents/trace32/t32-linux-awareness/
title: Trace32 Linux awareness
excerpt: "Trace32 linux awareness"
comments: false
toc: true
---


## Trace32 Linux Awareness

To implement Linux awareness,<br>
proceed sequentially with MMU setup, Task config, and Menu reprogramming.<br>

### MMU setup

<img src="/documents/images/trace32/t32linux0.png" alt="t32linux0" width="640" height="480"><br>
In <span style="{{ site.code }}">B::SYStem</span> , check MMUSPACES or use the command <span style="{{ site.code }}">B::SYStem.Option.MMUSPACES ON</span><br>
to enable the MMUSPACES option.<br>
<br>

<img src="/documents/images/trace32/t32linux1.png" alt="t32linux1" width="640" height="480"><br>
Use <span style="{{ site.code }}">B::y.list</span> to find the text symbol, which is the starting point of the code.<br>
The starting virtual address is <span style="{{ site.code }}">0xFFFF8000800000000</span> .<br>
<br>

<img src="/documents/images/trace32/t32linux2.png" alt="t32linux2" width="640" height="480"><br>
Find the end address of the virtual address, which is <span style="{{ site.code }}">0xFFFF800082030000</span> .<br>
<br>

<img src="/documents/images/trace32/t32linux3.png" alt="t32linux3" width="640" height="480"><br>
Use the <span style="{{ site.code }}">B::MMU.List.PageTable</span> or <span style="{{ site.code }}">B::MMU.List.EL1PageTable</span> command to check the MMU VA-to-PA mapping table.<br>
You can also see why there’s a slight difference from the text virtual address <B>(0xFFFF8000800000000)</B>.<br>
This is because it’s not the exact starting address.<br>
<br>

<img src="/documents/images/trace32/t32linux5.png" alt="t32linux5" width="640" height="480"><br>
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
B::TRANSlation.Create 0xFFFF8000000000000—0xFFFF80008202FFFF a:0x1A000D000
```
<br>

Since manually entering these addresses can be difficult,<br>
referencing symbols works just as well:
```
B::TRANSlation.Create (\\vmlinux\Global\_text)--(\\vmlinux\Global\_end-1) a:0x1A000D000
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
B::TRANSlation.Create (\\vmlinux\Global\_text)--(\\vmlinux\Global\_end-1) A:0x1A000D000
B::TRANSlation.COMMON 0xFFFF000000000000—0xFFFFFFFFFFFFFFFF
B::TRANSlation.TableWalk ON
B::TRANSlation.ON
```
<br>

With this, the MMU setup is complete.<br>
While MMU setup is complex, TASK setup and MENU addition are not as difficult.<br>
<br>

### Task config

<img src="/documents/images/trace32/t32linux6.png" alt="t32linux6" width="640" height="480"><br>
<img src="/documents/images/trace32/t32linux7.png" alt="t32linux7" width="640" height="480"><br>
Using <span style="{{ site.code }}">B::TASK.CONFIG {task file}</span> to proceed.<br>
Just like when loading an ELF file,<br>
you can either directly specify the file path or use \* to select the file manually.<br>
For reference, in the image, the file was written with a relative path.<br>
Using <span style="{{ site.code }}">B::cd</span> will display the current path where the program is running.<br>
You can confirm that a new ‘linux’ toolbar has been created.<br>
<br>

<img src="/documents/images/trace32/t32linux8.png" alt="t32linux8" width="640" height="480"><br>
Using <span style="{{ site.code }}">B::MENU.REPROGRAM {menu file}</span> to proceed.<br>
Just like when loading an ELF file,<br>
you can either directly specify the file path or use \* to select the file manually.<br>
For reference, in the image, the file was written with a relative path.<br>
Using <span style="{{ site.code }}">B::cd</span> will display the current path where the program is running.<br>
<br>

### Summary

To summarize all the commands for Linux awareness,
```
B::MMU.FORMAT LINUXSWAP3 \\vmlinux\Global\swapper_pg_dir
B::TRANSlation.create (\\vmlinux\Global\_text)--(\\vmlinux\Global\_end-1) a:0x1A000D000
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

<img src="/documents/images/trace32/t32linux9.png" alt="t32linux9" width="640" height="480"><br>
Like this, you can check the currently running processes,<br>
<br>

<img src="/documents/images/trace32/t32linux10.png" alt="t32linux10" width="640" height="480"><br>
Like this, you can also check kernel messages.<br>
<br>
