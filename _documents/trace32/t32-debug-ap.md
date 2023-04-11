---
permalink: /documents/trace32/t32-debug-ap/
title: Trace32 debug setting on AP
excerpt: "Trace32 debug setup (AP)"
comments: false
toc: true
---

## Trace32 debug

Trace32 Debugging Environment in this posting
```
The host PC environment is a Windows 11 (x86 PC).
The target board environment is an ARM-A Linux kernel 6.12.
```
<br>

### CPU Attach

<img src="/documents/images/trace32/t32debug0.png" alt="t32debug0" width="640" height="480"><br>
Start with the chip attached.<br>
Since chip verification is mostly register-based,<br>
there’s no need to load symbols and follow the code, but if debugging is required during kernel operation,<br>
you need to load symbols and track the code.<br>
<br>

### Program Counter

<img src="/documents/images/trace32/t32debug1.png" alt="t32debug1" width="640" height="480"><br>
Using the <span style="{{ site.code }}">B::Data.List</span> command allows you to check the address currently pointed to by the PC.<br>
The <span style="{{ site.code }}">B::Data.List</span> toolbar includes <B>Step</B>,<B>Over</B>, <B>Diverge</B>, <B>Return</B>, <B>Up</B>, <B>Go</B>,<B>Break</B>, and Mode operations.<br>
Let’s narrow it down to a few commonly used ones:<br>
<br>
Step executes the code while incrementing the PC by 4 each time.<br>
Over executes the code by skipping over function units.<br>
Up completes the execution of the currently entered function and reduces the stack depth by one level.<br>
Mode lets you choose whether to break it down into assembly code units.<br>
When broken down into assembly code units, it displays the code in a form modified by the compiler’s optimizations.<br>
This can be utilized after loading symbols.<br>
<br>

### Upload Symbol

<img src="/documents/images/trace32/t32debug2.png" alt="t32debug2" width="640" height="480"><br>
<img src="/documents/images/trace32/t32debug3.png" alt="t32debug3" width="640" height="480"><br>
Symbol registration is done by executing the <span style="{{ site.code }}">B::Data.LOAD.ELF {elf file} </span> command.<br>
The command supports auto-completion with the ‘Tab’ key, making it convenient to use.<br>
You can manually enter the file path, but using \* opens the file explorer, allowing you to select it directly.<br>
<br>

<img src="/documents/images/trace32/t32debug4.png" alt="t32debug4" width="640" height="480"><br>
If you simply select the file, an error will likely occur.<br>
Since this is done without uploading the code, use the <B>/nocode</B> option.<br>

<img src="/documents/images/trace32/t32debug5.png" alt="t32debug5" width="640" height="480"><br>
Once the symbols are successfully loaded, you can view the symbol list like this.<br>
You can see the symbol list by clicking the blue exclamation mark on the GUI,<br>
or by using <span style="{{ site.code }}">B::sYmbol.Browse.sYmbol</span> or <span style="{{ site.code }}">B::y.list</span> .<br>
If the symbol you’re looking for isn’t visible, you need to expose the symbols during the kernel build.<br>
(Usually, removing "static" is enough to expose the symbols.)<br>
Now, with the B::Data.List command, you can check the code currently pointed to by the PC.<br>
<br>

<img src="/documents/images/trace32/t32debug6.png" alt="t32debug6" width="640" height="480"><br>
However, if the code doesn’t display properly and is covered with diagonal lines,<br>
it’s because the symbols aren’t correctly matched. This issue occurs when the symbol path cannot be traced,<br>
and you need to realign the symbol path.<br>
This varies depending on each debugging environment;<br>
(in my case, I’m connected from a Windows PC to a remote VM via a Samba server.)<br>
<br>

<img src="/documents/images/trace32/t32debug7.png" alt="t32debug7" width="640" height="480"><br>
First, right-click anywhere in the window and then select "View Info."<br>
<br>

<img src="/documents/images/trace32/t32debug8.png" alt="t32debug8" width="640" height="480"><br>
This displays the path where the symbol (ELF file) was stored when it was built.<br>
Check whether this path is traceable from the current host OS.<br>
In the current environment, since the host OS (Windows) cannot trace idle.c<br>
located on a VM connected via a Samba server, you need to make the path traceable.<br>
<br>

#### Resolve Path

<img src="/documents/images/trace32/t32debug9.png" alt="t32debug9" width="640" height="480"><br>
Right-click and select "Resolve Path."<br>
<br>

<img src="/documents/images/trace32/t32debug10.png" alt="t32debug10" width="640" height="480"><br>
You can resolve it by finding and selecting the source that the symbol actually points to.<br>

<img src="/documents/images/trace32/t32debug11.png" alt="t32debug11" width="640" height="480"><br>
You can confirm that the previously invisible code is now fully visible.<br>

#### Practice command 1

<img src="/documents/images/trace32/t32debug13.png" alt="t32debug13" width="640" height="480"><br>
Perform <span style="{{ site.code }}">B::sYmbol.RESet</span> and repeat the process up to the point right after loading the symbols again.<br>
Use the <span style="{{ site.code }}">B::sYmbol.SourcePATH.Translate {invalid path} {correct path}</span> command to change the untraceable path to a traceable one.<br>

<img src="/documents/images/trace32/t32debug8.png" alt="t32debug8" width="640" height="480"><br>
The "\home\B240691" part cannot be traced by the host OS.<br>
If you modify this part to the "Z:" path connected via the Samba server, it becomes traceable.<br>

<img src="/documents/images/trace32/t32debug12.png" alt="t32debug12" width="640" height="480"><br>
<img src="/documents/images/trace32/t32debug11.png" alt="t32debug14" width="640" height="480"><br>
With the command <span style="{{ site.code }}">B::sYmbol.SourcePATH.Translate "/home/B240691" "Z:"</span> ,<br>
the symbol matching has been successfully completed.<br>
<br>

#### Practice command 2

<img src="/documents/images/trace32/t32debug13.png" alt="t32debug13" width="640" height="480"><br>
There’s also a method to align the path from the start when loading the symbols.<br>
<br>

<img src="/documents/images/trace32/t32debug14.png" alt="t32debug14" width="640" height="480"><br>
<img src="/documents/images/trace32/t32debug11.png" alt="t32debug11" width="640" height="480"><br>
With the command
```
B::d.load.elf Z:\work1\tcc807x\main\kernel-6.12\vmlinux /nocode /path Z:\work1\tcc807x\main\kernel-6.12 /StripPART "kernel-6.12"
```
you can load the symbols while aligning the path from the start.<br>
<br>

The moment vmlinux is loaded,<br>
the symbol path stored in the symbol file is registered in the debugging program,<br>
and this is overridden with the <B>/path</B> option.<br>
Then, the "kernel-6.12" portion of this path is trimmed using the <B>/StripPART</B> option.<br>
<br>

### Break Point

<img src="/documents/images/trace32/t32debug15.png" alt="t32debug15" width="640" height="480"><br>
Use B::Break to halt the CPU.<br>
<br>

<img src="/documents/images/trace32/t32debug16.png" alt="t32debug16" width="640" height="480"><br>
Select the Break Point option from the top menu, or execute the <span style="{{ site.code }}">B::Break.List</span> command.<br>
<br>

<img src="/documents/images/trace32/t32debug17.png" alt="t32debug17" width="640" height="480"><br>
Open the symbol window and search for the symbol you want to set.<br>
If a function is not exposed as a symbol, it cannot be found. In that case,<br>
you need to expose the symbol, rebuild the code, and upload it again.<br>
<br>

<img src="/documents/images/trace32/t32debug18.png" alt="t32debug18" width="640" height="480"><br>
<img src="/documents/images/trace32/t32debug19.png" alt="t32debug19" width="640" height="480"><br>
<br>
For example, if you want to set a breakpoint inside the GIC handler at <span style="{{ site.code }}">gic_handle_irq</span> ,<br>
search for the <span style="{{ site.code }}">gic_handle_irq</span> symbol.<br>
Once found, you can double-click the symbol to set a breakpoint.<br>
<br>

<img src="/documents/images/trace32/t32debug20.png" alt="t32debug20" width="640" height="480"><br>
Select "To Command Line..." to switch to command mode.<br>
<br>

<img src="/documents/images/trace32/t32debug21.png" alt="t32debug21" width="640" height="480"><br>
It is recommended to change the breakpoint method to <B>/Onchip</B> before use.<br>
The default is <B>/SOFT</B>, which relies on a software fallback mechanism,<br>
So breakpoint information may be lost.<br>
<br>
If you use the <B>/Onchip</B> option,<br>
the Virtual Address is stored in the ARMv8-A breakpoint registers( <span style="{{ site.code }}">DBGBVRn_EL1</span> and <span style="{{ site.code }}">DBGBCRn_EL1</span> ).<br>
<br>

<img src="/documents/images/trace32/t32debug22.png" alt="t32debug22" width="640" height="480"><br>
Due to the interrupt-driven nature of the Linux kernel,<br>
a breakpoint will be hit almost immediately after executing <span style="{{ site.code }}">B::Go</span><br>
<br>

<img src="/documents/images/trace32/t32debug23.png" alt="t32debug23" width="640" height="480"><br>
The PC (Program Counter) stopped at the <span style="{{ site.code }}">gic_handle_irq</span> section.<br>
<br>

<img src="/documents/images/trace32/t32debug24.png" alt="t32debug24" width="640" height="480"><br>
By selecting Mode, you can also analyze atomic operations.<br>
(Note: Since the code is optimized into assembly, the actual layout may differ from the source code.)<br>
When debugging by stepping through the code,<br>
<span style="{{ site.code }}">Step</span> executes one command at a time depending on the selected Mode.<br>
<span style="{{ site.code }}">Over</span> executes code line by line.<br>
<span style="{{ site.code }}">Up</span> executes all the depth at once, that is, executes all the first callers.<br>
<span style="{{ site.code }}">Go</span> and <span style="{{ site.code }}">Break</span> are used to release or hold the CPU.<br>
<br>

<img src="/documents/images/trace32/t32debug25.png" alt="t32debug25" width="640" height="480"><br>
<img src="/documents/images/trace32/t32debug26.png" alt="t32debug26" width="640" height="480"><br>
You can check the code stack using <span style="{{ site.code }}">B::Frame</span> You can navigate through the stack using Up and Down.<br>
Locals and Caller can be viewed by adding options to the command or by selecting them directly.<br>
<br>

<img src="/documents/images/trace32/t32debug27.png" alt="t32debug27" width="640" height="480"><br>
Additionally, breakpoints can be set with conditions.<br>
<br>
