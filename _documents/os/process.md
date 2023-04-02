---
permalink: /documents/os/process/
title: Process
toc: true
---

## Chap 2. Process

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
  <img src="/assets/images/os/process/isolation.png" alt="isolation" width="640" height="480"><br>
  <span style="{{ site.en }}">picture [1] Process isolation</span>
</p>
<br>

#### One program, multiple processes

The state in which a program runs is called a process, so does one program have only one process?

The answer is yes or no.<br>
It can be one, it can be more than two.<br>
A program with multiple processes in one program.<br>

<p align="center">
  <img src="/assets/images/os/process/data.png" alt="data" width="640" height="480"><br>
  <span style="{{ site.en }}">picture [2] 1 Program with n process</span>
</p>
<br>

Program code is held together in one program, and process-specific data is held in process units.

#### process elements with pcb

A process has various properties.
The set of these properties is called a "Process Control Block."

<p align="center">
  <img src="/assets/images/os/process/pcb.png" alt="pcb" width="640" height="480"><br>
  <span style="{{ site.en }}">picture [3] 1 Program n process</span>
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
  <img src="/assets/images/os/process/context-switching.png" alt="context-switching" width="640" height="480"><br>
  <span style="{{ site.en }}">picture [4-1] Context switching</span>
</p>
<br>

<p align="center">
  <img src="/assets/images/os/process/dispatcher.png" alt="dispatcher" width="640" height="480"><br>
  <span style="{{ site.en }}">picture [4-2] Dispatcher</span>
</p>
<br>

As we checked in Overview, OS is a program.<br>
Called when context switching is required.<br>

#### process state

Process state model.<br>
Nothing is fixed, efficiency is important.<br>

<p align="center">
  <img src="/assets/images/os/process/state.png" alt="state" width="640" height="480"><br>
  <br>
  <span style="{{ site.en }}">picture [5] process state model</span>
</p>
<br>

| State | Info |
|:---:|:---|
| Ready | The program is waiting. Process state that can be in the "Running" state<br>**when only CPU is assigned.** |
| Running | The operational. The Ready/Waiting state occurs when the time is up,<br>an interrupt occurs, or other demands are required. |
| Waiting(=blocked) | The standby state requires other resources, including the CPU. |
| Suspend | Processes that are queued have been transferred to the HDD due to infrequently used processes or memory over due to processes. |

