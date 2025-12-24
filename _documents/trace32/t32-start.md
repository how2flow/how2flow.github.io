---
permalink: /documents/trace32/t32-start/
title: Trace32 Install & Start
excerpt: "Start Trace32 Debugger"
comments: false
toc: true
---

## Trace32 start

<img src="/assets/images/documents/trace32/t32start0.png" alt="t32start0" width="640" height="480"><br>
When installing, the program is set to be installed under the C:\T32 by default.<br>
It’s a good idea to rename the folder according to the version during installation.<br>
This is because, regardless of the version, the default installation path for all programs is C:\T32,<br>
and when the program runs, it searches for files in the installation directory.<br>
If you have more than one version installed and in use, you’d need to rename the folder to "T32" every time you run a different version.<br>
Since I use both the 2017 and 2023 versions,<br>
I renamed the folders during installation to T32_2017 and T32_2023,<br>
depending on the version. The reason for using multiple versions will be explained later.<br>
<br>

<img src="/assets/images/documents/trace32/t32start1.png" alt="t32start1" width="640" height="480"><br>
Connect the Trace32 debugger to the JTAG of the board and plug the USB into the PC.<br>
You should confirm that the device is recognized by the PC.<br>
<br>

<img src="/assets/images/documents/trace32/t32start2.png" alt="t32start2" width="640" height="480"><br>
<img src="/assets/images/documents/trace32/t32start3.png" alt="t32start3" width="640" height="480"><br>
You can start the program by running <span style="{{ site.code }}">C:\T32_2023\bin\t32m{arch}.exe</span> , or you can launch it from the Start menu using "Trace32 Start."<br>
Since "Trace32 Start" also requires selecting the architecture, I usually prefer the former method.<br>
<br>

### Check License

<img src="/assets/images/documents/trace32/t32start4.png" alt="t32start4" width="640" height="480"><br>
<img src="/assets/images/documents/trace32/t32start5.png" alt="t32start5" width="640" height="480"><br>
On the initial screen after launching, you can click <B>[Help] -> [About]</B> Trace32<br>
to check the program version information and license details.<br>
In this case, the Software version is 09/23, and the debugger License is valid until 06/2025.<br>
If the Software version is newer than the guaranteed License period, it will be used as a trial version.<br>
In such cases, the program will terminate after 10 minutes.<br>
This is why I install and use more than one version of the program.<br>
<br>

### Help

<img src="/assets/images/documents/trace32/t32start6.png" alt="t32start6" width="640" height="480"><br>
<img src="/assets/images/documents/trace32/t32start7.png" alt="t32start7" width="640" height="480"><br>
Trace32 is very well-documented, so most information can be searched for and found through <B>[Help] -> [Find]</B>.<br>
However, the sheer volume of information can make it a skill to locate exactly what I need.<br>
<br>

### Basic menu

<img src="/assets/images/documents/trace32/t32start8.png" alt="t32start8" width="640" height="480"><br>
In the File menu, I mainly use the functions for creating or loading scripts.<br>
The debugging program can also perform all operations based on commands.<br>
You can create a new script by selecting "New Script" or use the B::pedit command to create a new script.<br>
<br>

<img src="/assets/images/documents/trace32/t32start9.png" alt="t32start9" width="640" height="480"><br>
When selecting a menu through the GUI, you may notice options like <span style="{{ site.code }}">B::pedit</span> written in the menu window.<br>
Ultimately, this executes by calling a command, and you can enter commands starting with <span style="{{ site.code }}">B::</span> in the input window below.<br>
<br>

<img src="/assets/images/documents/trace32/t32start10.png" alt="t32start10" width="640" height="480"><br>
If you right-click the frame of a new window,<br>
the command will be auto-completed, and you can add further input as well.<br>
<br>

### Setup target

<img src="/assets/images/documents/trace32/t32start11.png" alt="t32start11" width="640" height="480"><br>
Now, use the attach script provided by the vendor to connect the board to the debugger program.<br>
While you can configure detailed settings manually,<br>
it’s impossible to do so without the Corebase and CTIbase information being provided.<br>
<br>

<img src="/assets/images/documents/trace32/t32start12.png" alt="t32start12" width="640" height="480"><br>
<img src="/assets/images/documents/trace32/t32start13.png" alt="t32start13" width="640" height="480"><br>
You can go to <B>[CPU] -> [System Settings]</B> to modify core information, or, as shown in the frame,<br>
you can use the command <span style="{{ site.code }}">B::SYStem</span> to bring up that menu.<br>
Commands are not case-sensitive, and the parts written in uppercase can be used as abbreviations.<br>
For example:
```
SYStem == sys / sYmbol == y
```
<br>

<img src="/assets/images/documents/trace32/t32start14.png" alt="t32start14" width="640" height="480"><br>
There are various system modes, and by connecting the debugger with "attach" and setting a breakpoint with B::break,<br>
you can dump and view the memory area like this.<br>
The principle is to break the CPU and view the memory through the CPU.<br>
The "AHD" in front of the address indicates the class: <span style="{{ site.code }}">A</span> stands for physical address, <span style="{{ site.code }}">H</span> stands for EL2, and <span style="{{ site.code }}">D</span> stands for data type.<br>
By prefixing the class with "a," you can access registers mapped to the physical address memory.<br>
<br>
<B><i>So, how do you access architecture system registers?</i></B><br>
<br>

<img src="/assets/images/documents/trace32/t32start15.png" alt="t32start15" width="640" height="480"><br>
ARM-A provides a tool to access system registers encoded in this way. (As of the 2023 version)<br>

<img src="/assets/images/documents/trace32/t32start16.png" alt="t32start16" width="640" height="480"><br>
You just need to find and access the register address you need.<br>
Alternatively, as shown in the tool, you can access it using the "spr" class<br>
(e.g., B::d.dump spr:0x33e21).<br>
