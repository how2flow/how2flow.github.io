---
permalink: /documents/os/process/
title: Process
excerpt: "What is the process? What's its role in os?"
toc: true
---

## Process

### process

process is a instance of running program.<br>
program is command list.<br>
All resources are distributed on a per-process basis.<br>

#### process isolation

The process must be independent.<br>
From a process perspective, you should think that a process has all of its resources and that they are all available.<br>
The CPU splits time to ensure isolation,<br>
Memory guarantees isolation by dividing space.<br>

<p align="center">
  <img src="/documents/images/os/process/isolation.png" alt="isolation" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [1] Process isolation</span>
</p>
<br>

#### One program, multiple processes

The state in which a program runs is called a process, so does one program have only one process?

The answer is yes or no.<br>
It can be one, it can be more than two.<br>
A program with multiple processes in one program.<br>

<p align="center">
  <img src="/documents/images/os/process/data.png" alt="data" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [2] 1 Program with n process</span>
</p>
<br>

Program code is held together in one program, and process-specific data is held in process units.

#### process elements with pcb

A process has various properties.
The set of these properties is called a "Process Control Block."

<p align="center">
  <img src="/documents/images/os/process/pcb.png" alt="pcb" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [3] 1 Program n process</span>
</p>

| Elements | Info |
|:---:|:---|
| Identifier | process ID (PID) |
| State | ready, running waiting ... |
| Priority | priority of process |
| Context data | Space for backup/recovery of information<br>about processes immediately before/after, and after a process is passed |
| Memory data | data |
| Program Counter | Space to store the address of the next command to be executed |
| I/O status | About I/O devices |
| Accounting infomation | ... |

#### context switching

Context switching is the process of implementing time-sharing<br>
and continuously operating from the perspective of the process.<br>
<br>
Backing up the current process status to a block of context and recovering the block of context<br>
for the next process is called context switching when the allocated time is over.<br>
Context switching is done by the Dispatcher(OS).<br>

<p align="center">
  <img src="/documents/images/os/process/context-switching.png" alt="context-switching" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [4-1] Context switching</span>
</p>
<br>

<p align="center">
  <img src="/documents/images/os/process/dispatcher.png" alt="dispatcher" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [4-2] Dispatcher</span>
</p>
<br>

As we checked in Overview, OS is a program.<br>
Called when context switching is required.<br>

#### process state

Process state model.<br>
Nothing is fixed, efficiency is important.<br>

<p align="center">
  <img src="/documents/images/os/process/state.png" alt="state" width="640" height="480"><br>
  <br>
  <span style="{{ site.img }}">picture [5] process state model</span>
</p>
<br>

| State | Info |
|:---:|:---|
| Ready | The program is waiting. Process state that can be in the "Running" state<br>**when only CPU is assigned.** |
| Running | The operational. The Ready/Waiting state occurs when the time is up,<br>an interrupt occurs, or other demands are required. |
| Waiting(=blocked) | The standby state requires other resources, including the CPU. |
| Suspend | Processes that are queued have been transferred to the HDD due to infrequently used processes or memory over due to processes. |

### Tables

The OS has several tables to manage the process.<br>

<p align="center">
  <img src="/documents/images/os/process/tables.png" alt="tables" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [6] Tables</span>
</p>
<br>

#### Memory table

Memory table has main memory for running programs, virtual memory and protection attributes.<br>
Virtual memory will be covered in more detail in <a href="/documents/os/memory/">here</a><br>

#### I/O table

Exists to deceive that the process has a complete monopoly on I/O.<br>
The OS needs to know the status of the I/O and the source and purpose of the I/O.<br>
Hardware interruptions occur in I/O.<br>

#### File table

The file address must be saved separately for the process to access the file and perform operations such as read/write.<br>
It is stored separately in the file table, also known as the **F**ile **A**llocation **T**able, Each OS has its own way.<br>
Recently, many **NTFS** methods have been adopted, but this is the traditional method of storing files on hard disks.<br>

<p align="center">
  <img src="/documents/images/os/process/hdd.png" alt="hdd" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [7] FAT</span>
</p>
<br>

Memory is too large to be managed per address in bytes (address).<br>
This picture shows saving the file in blocks.<br>
The "memo.txt" file is split into blocks and stored separately.<br>
The OS uses FAT to find data and use files.<br>
In Linux, it is also called a file system.<br>

### Struct of Process

#### Memory

The process consists largely of attributes and locations.<br>

<p align="center">
  <img src="/documents/images/os/process/struct.png" alt="struct" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [8] Process Struct</span>
</p>
<br>

code is read-only layer. ('text')<br>
Global, static, array, structure vars, and so on are stored in the data area. (+ bss)<br>
User functions are in stack area. It gets bigger or smaller by its user functions.<br>

<p align="center">
  <img src="/documents/images/os/process/phym.png" alt="phym" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [9] Virtual memory vs Physical memory</span>
</p>
<br>

It's actually not saved as pretty.<br>
Don't be confused with the logical image.<br>

#### Process creation & interrupt

A new process is created by the needs of the OS.<br>

```
1. Assigns Process ID
2. Assigns memory
3. Initialize PCB
4. Link functions: win: (*.dll) linux: (*.so) ...
5. Data Structuring & Scaling
```

An **interrupt** occurs and another process is created/executed or terminated.<br>
<br>
Interrupt includes<br>
```
- exception
- interrupt (H/W)
- software interrupt (os)
```

The exception is an real unexpected move.<br>
(poweroff, div with 0 etc ...)<br>
<br>
interrupt is related to hardware I/O operation.<br>
(Print complete, timer(scheduling), etc ...)<br>
<br>
software interrupt is a call from the OS.<br>
It usually happens when `context-change` or change `permissions`.
When a software interrupt is operated by an interrupt handler(ISR), a system call (or supervisor call) operates.<br>

#### Permissions: user mode vs system mode

user mode: Minimum permissions not affecting the system.<br>
system mode: All rights on the system. <br>

##### system(kernel) mode

Process Management

```
Process creation and termination # process isolation
Process scheduling and dispatching
Process switching # context switching
Process syncronization and support for interprocess comunication # IPC
Management of Process Control Blocks
```
<span style="{{ site.code }}">IPC</span>: There are times when resource sharing between processes is necessary.<br>

Memory Management

```
Allocation of address space to process
Swapping
Page and segment management
```
<span style="{{ site.code }}">Page</span> , <span style="{{ site.code }}">segment</span> - related content will be posted <a href="/documents/os/memory">here.</a><br>

I/O Management

```
Buffer Management # buffer is the memory of devices.
Allocation of I/O channels and devices to process
```

Support functions

```
Interrupt handling
Accounting
Monitoring
```
