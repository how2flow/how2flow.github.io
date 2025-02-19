---
permalink: /documents/trace32-linuxawareness/
title: Trace32
excerpt: "Trace32 cmm script"
comments: false
toc: true
---

## Trace32 Linux awareness

TRACE32's Linux Awareness provides advanced debugging capabilities specifically tailored for Linux-based systems.<br>
It enables developers to debug the Linux kernel, modules, and user-space applications with deep insights into the operating system.<br>

<B>Key features include:</B><br>
```
1. Kernel Debugging: Access kernel structures, variables, and task states directly.
2. Process Awareness: View and debug individual processes, including their threads, memory, and execution states.
3. Symbolic Debugging: Use symbols and source code for kernel and application debugging.
4. Task-Specific Breakpoints: Set breakpoints on specific tasks or threads for targeted debugging.
5. MMU Support: Handle virtual-to-physical address translations seamlessly.
6. File System and IPC Awareness: Debug file operations and inter-process communication mechanisms.
```
<br>
These capabilities streamline debugging and profiling for embedded Linux systems,<br>
making TRACE32 an essential tool for Linux developers.<br>

### How to

After attaching the TRACE32 debugger to the core, a few additional steps are required.<br>
The attach script is typically provided by the vendor or can be found in the chip's datasheet.<br>
The attachment must be performed in an environment where the Linux kernel has already booted.<br>
<br>
First, upload the Linux ELF file.<br>
Next, enable the MMUSPACE option in the debugger.<br>
Finally, register the Linux awareness configuration and menu provided by the TRACE32 PowerView program.
```
B:: Data.LOAD.elf "path_of_vmlinux" /NoCODE                       ; ... line 1
B:: sYmbol.sourcePATH "invalid_part" "correct_part"               ; ... line 2
B:: SYStem.OPTION MMUSPACES ON                                    ; ... line 3
B:: task.CONFIG * // or "~~/demo/arm/kernel/linux/awareness/linux3.t32"   ; ... line 4
B:: MENU.ReProgram * // or "~~/demo/arm/kernel/linux/awareness/linux.men" ; ... line 5
B:: if !run() go
B:: break
```
```
line 1: Load linux elf for reading symbols
line 2: This is used only when the target ELF file is stored in a remote environment, such as NFS or Samba
line 3: MMUSPACES option ON
line 4: Use linux awareness config file
        The parameter example assumes the default execution path of PowerView is located at "T32\bin\windows64"
        You can verify the actual path in your environment using the [B:: cd] command.
line 5: Use linux menu config file
        The parameter example assumes the default execution path of PowerView is located at "T32\bin\windows64"
        You can verify the actual path in your environment using the [B:: cd] command.
```
<br>
