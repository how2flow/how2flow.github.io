---
permalink: /documents/os/overview/
title: Overview
toc: true
---

## Chap1. Overview

This post is an overview. Just think about it like "Most of them have this and that" and move on.<br>
The concepts shown here will be covered in detail in a later posting.<br>

---

Anyone who has used a computer has heard the word OS.<br>
Windows, macOS, Linux, etc., which we've heard about once in our lives, are all operating systems.<br>
<br>
Why do I need an OS? It is necessary for resource management.<br>
ls I will talk about OS in the next, resources(cpu, memory, etc ...) are limited and very important.<br>
First, I will briefly explain the **computer system** and **OS**.<br>

### Computer System

Before we get into the OS, let's take a look at the CPU and resister that are at the heart of the computer.<br>
The CPU consists largely of ALU, registers, and I/Operipherals. (Three Elements of CPU)<br>
<br>

Register:<br>
Resistors are architecturally distinct, but are mostly divided into regular and special registers.<br>
Special registers include PC (Program Counter), IR (Instruction Register), MAR (Memory Address Register), and MBR (Memory Buffer Register).<br>
<br>

Memory:<br>
Memory has layers.
```
                        (speed)
    cpu           |        ^
-------------     |        |
cache memory      |        |
-------------     |        |
 main memory      |        |
-------------     |        |
    HDD           v        |
             (capacity)
```
The closer it is to the CPU, the faster it is, and the closer it is to the HDD (the more SSD or NVME is used in parallel these days), the slower it is and the larger the capacity.<br>

This is a summary of the CPU and Computer.
```
<Computer>

+-------------------------- CPU ---------------------------+   +-------- Memory --------+
|                                                          |   |                        |
| +----- ALU -----+                                        |   |                        |
| |               |                                        |   |                        |
| | + / - / * ... |                                        |   |                        |
| |               |                                        |   |                        |
| +---------------+                                        |   |                        |
|         ^ |                                              |   |                        |
|         | |                                              |   |                        |
|         | v                                              |   +------------------------+
| +--- Register ---+                +--- Control Unit ---+ |
| |                |                |                    | |   +---- I/O Peripherals ---+
| |                | <------------  |                    | |   |                        |
| |                | ------------>  |                    | |   |                        |
| |                |                |                    | |   |                        |
| +----------------+                +--------------------+ |   |                        |
|                                                          |   |                        |
+----------------------------------------------------------+   +------------------------+
```
The computer consists largely of CPU, main memory, and I/O peripherals,<br>
and the CPU consists of ALU, a memory area register, and a control unit.<br>

### Operating System

OS is an interface between application and hardware.<br>

If you look at the overall structure briefly,

```
API
---\
    application
ABI
---\
    OS
ISA
---\
    Hardware
```
It can be represented like above.

#### Upgrade steps of OS

I'll describe the concept of OS with the development steps of OS.<br>

| step | Index |
|:---:|:---:|
| step 5| multi physics (multi core)|
| step 4| time-sharing|
| step 3| multi programming batch system|
| step 2| simple batch system|
| step 1| serial process|

---

**Step 1. serial process**

It's the beginning of the process, It is not Operating System.<br>
<br>

**Step 2. simple batch system**

It is a method of sequentially processing programs over time.<br>
```

+--- monitor ---+                 +--- monitor ---+                 +--- monitor ---+
|               |                 |               |                 |               |
|               | => program A => |               | => program B => |               | => ...
|               |                 |               |                 |               |
+---------------+                 +---------------+                 +---------------+

```

These are the conditions for the desirable behavior shown at this stage.<br>
- monitor memory protection.
- Timer.
- privileged Instruction. (user mode / kernel mode)
- Interrupt.
<br>

**Step 3. multi programming batch system.**

It's to keep various programs running nonstop.<br>
How multiple user programs are handled as if they were running simultaneously.<br>
The concept of <span style="{{ site.important }}">concurrency</span> emerges.<br>
<span style="{{ site.important }}">Parallel processes of multi core and concurrency are different.</span><br>

```
case 1. [A] ...wait... [B] ...wait... [C] ...
case 2. .. [D] ...wait... [E] ...wait... [F] ...

...

======> [A][D] ....... [B][E] ....... [C][F] ...

```
<br>

**Step 4. time-sharing**

Improved user responsiveness over multi-deployment approaches.<br>
In general, each program sets its own time and runs only that amount of time and moves on to the next program.<br>
Typically, there is a round-robin method.<br>
<br>

**Step 5. multi physics (multi core)**

It literally means multiple cores.<br>
OS documentation is based on <span sytle="{{ site.important }}">single core.</span><br>

<a href="{{ site.baseurl }}/documents/os-process/">Next =></a><br>
