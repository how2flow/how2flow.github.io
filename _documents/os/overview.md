---
permalink: /documents/os/overview/
title: Overview
excerpt: "Operating System Overview"
toc: true
---

## Overview

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

Before we get into the OS, let's take a look at the CPU and computer structure.<br>
These are structure summaries of the CPU and Computer.<br>

<p align="center">
  <img src="/documents/images/os/overview/computer.png" alt="computer" width="640" height="480"><br>
  <br>
  <span style="{{ site.img }}">picture [1-1] Computer structure</span>
</p>
<p align="center">
  <img src="/documents/images/os/overview/cpu.png" alt="cpu" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [1-2] CPU structure</span>
</p>
<br>

The computer consists largely of CPU, main memory, and I/O modules,<br>
and the CPU consists of ALU, a memory area register, and a control unit.<br>
Various inputs and commands are passed through the bus to memory and registers, registers and ALUs, and many other devices.<br>
<br>

The memory is hierarchical.<br>
<p align="center">
  <img src="/documents/images/os/overview/memory-hierarchy.png" alt="hierarchy" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [2] Memory hierarchy</span>
</p>
<br>

The closer it is to the CPU, the faster it is, and the closer it is to the HDD (the more SSD or NVME is used in parallel these days), the slower it is and the larger the capacity.<br>


<span style="font-size:60%">**good terms to know**</span><br>
<span style="font-size:60%">- GPU: **G**raphical **P**roccessing **U**nit.<br>It is equipped with hundreds or thousands of cores.</span><br><br>
<span style="font-size:60%">- SIMD: **S**ingle **I**nstruction **M**ulti **D**ata.<br>It is a type of parallel processing that enables a single instruction to be executed simultaneously on multiple pieces of data.</span><br><br>
<span style="font-size:60%">- DSP: **D**igital **S**ignal **P**rocessor.<br>It is a specialized microprocessor designed for processing digital signals such as audio, video, and telecommunications signals.</span><br><br>


### Operating System

There's one I need to make sure before start os.<br>
OS is an omnipotent system, but it's like a program.<br>

<div style="display:flex; justify-content:center; align-items:center;">
  <img src="/documents/images/os/overview/call.png" alt="call" width="320" height="320" style="margin: 0 80px;">
  <img src="/documents/images/os/overview/command.png" alt="command" width="320" height="320" style="margin-right:80px;">
</div>
<div style="display:flex; justify-content:center;">
  <span style="margin: 0 240px 0 0; font-size:60%">picture [3-1] OS is a program</span>
  <span style="margin: 0 0 0 0; font-size:60%">picture [3-2] command flow</span>
</div>
<br>

The command, flow, and fetch will be explained in <a href="/documents/os/process/">"Chap2. process",</a><br>
but what is dispatched is that the OS is called between programs as well as regular programs to manage resources.<br>

OS is an interface between application and hardware.<br>

If you look at the overall structure briefly,

| interface | full-name | for |
|:---:|:---:|:---:|
| API | **A**pplication **P**rogramming **I**nterface | app |
| ABI | **A**pplication **B**inary **I**nterface | os |
| ISA | **I**nstruction **S**et **A**rchitecture | hardware |

<span style="margin: 0 0 0 160px; {{ site.img }}">table [1] library</span>
<br><br>

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

<span style="margin: 0 0 0 60px; {{ site.img }}">table [2] steps of upgrade</span>
<br>

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
The concept of <span style="color:red">concurrency</span> emerges.<br>
<span style="{{ site.important }}">Parallel processes of multi core and concurrency are different.</span><br>

**Step 4. time-sharing**

Improved user responsiveness over multi-deployment approaches.<br>
In general, each program sets its own time and runs only that amount of time and moves on to the next program.<br>
Typically, there is a round-robin method.<br>
<br>

**Step 5. multi physics (multi core)**

It literally means multiple cores.<br>
OS documentation is based on <span sytle="{{ site.important }}">single core.</span><br>

<a href="{{ site.baseurl }}/documents/os/process/">Next =></a><br>
