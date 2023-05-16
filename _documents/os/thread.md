---
permalink: /documents/os/thread/
title: Thread
excerpt: "What is the thread? What's its role in os?"
toc: true
---

## Thread

thread is Like a process, it means one execution unit.<br>
A unit smaller than a process.<br>
It's a split in the process.<br>

### Process vs Thread

Prochess is the unit of resource(cpu, memory, I/O ...) ownership.<br>
Thread is the unit of dispatching.<br>

Process switching is called context switching.<br>
Thread exchange is also called context switching.<br>
If there is a difference, switching between threads occurs very quickly.<br>

<p align="center">
  <img src="/documents/images/os/thread/process_vs_thread.png" alt="p_vs_t" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [1] Threads in Process</span>
</p>
<br>

Resource sharing between processes is not possible.<br>
However, it is possible to share resources between threads within the same process.<br>

IPC should be used for interaction between processes.<br>

#### multi threading

The ability of an OS to support multiple, concurrent paths of execution with in a single process.<br>

concurrency or parallel must be ensured.<br>
As I posted in <a href="/documents/os/overview/#upgrade-steps-of-os">Overview</a>, the two are different.<br>

<span style="{{ site.code }}">concurrency</span> is that Through time sharing and context switching,<br>
It runs as if it's working at the same time.<br>
It is not physically executed, but it can be said to be executed because it has not been terminated.<br>
<span style="{{ site.code }}">parallel</span> is real working at the same time with multi core.<br>

<span style="{{ site.code }}">PCB</span> : process control block<br>
<span style="{{ site.code }}">TCB</span> : thread control block<br>

<p align="center">
  <img src="/documents/images/os/thread/multithread.png" alt="multithread" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [2] multi thread model</span>
</p>
<br>

The PCB contains the TCB of each thread.<br>
Eanch thread has a <a href="http://0.0.0.0:4000/documents/os/process/#process-state">state</a> (running, ready, etc ...)
save context when not running.<br>
As mentioned above, they can access to the memory and resorces of its process.<br>

This is types of multithread model<br>

<p align="center">
  <img src="/documents/images/os/thread/type.png" alt="type" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [3] types of multithread</span>
</p>
<br>

1 Process & 1 thread is typically 'MS-DOS'.<br>
The concept and term <span style="{{ site.code }}">thread</span> did not exist because all actions were performed by the process.<br>

1 process & n thread is typically 'JVM'.<br>

```
+-----------+
|    JVM    |
+-----------+
|  windows  |
+-----------+
|    H/W    |
+-----------+
```
Java is because a virtual machine runs on Windows and acts on it.<br>

Classic OS are implemented in a 1 process & 1 thread structure in a multi-process.<br>
With the development of threads and multi-threads,<br>

it has the current form(Windows, Linux, etc ...) of multiple processes in the form of multi-threads of one process.<br>

#### benefits of thread

thread has more benefits than process.

```
- Take less time to create than process.
- Take less time to terminate a thread than process.
- Take less time to switching between threads than processes.
- Take less time to communicate between threads than processes.
```

#### thread characteristics

This is thread characteristics.

```
- foreground and background work(job).
- asyncronous(non blocking) processing.
- speed execution.
- modular program structure.
```

So In a OS that supports thread, scheduling and dispatching is done on a thread basis.<br>
Of course, When a process is suspended, all threads under it are suspended,<br>
and when the process is terminated, all threads under it are terminated.<br>


### RPC

**R**emote(server) **P**rocedure(function) **C**all<br>

<p align="center">
  <img src="/documents/images/os/thread/rpc.png" alt="rpc" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [4] Remote Procedure Call</span>
</p>
<br>

It is to retrieve the necessary resources from the server.<br>

### Thread syncronize

Synchronization is when the other starts at the end of one,<br>
and when the other ends, the other starts again.<br>

That is, every time a thread is passed, it is cut off, which is called blocking.<br>
Synchronization proceeds on with blocking.<br>
Asynchronization proceeds on its own without blocking.<br>

<p align="center">
  <img src="/documents/images/os/thread/sync.png" alt="sync" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [5] Sync vs Async</span>
</p>
<br>

Since threads in a process are connected, resource needs should not conflict.<br>

### ULT KLT

<span style="{{ site.code }}">ULT</span> :
**U**ser **L**evel **T**hread. (user mode)<br>
multi-core is compatible. (parallel)<br>
but, <span style="color: red">It should be implemented by the programmer.</span><br>
null-fork/signal wait is very fast.<br>

<p align="center">
  <img src="/documents/images/os/thread/mode1.png" alt="sync" width="320" height="240"><img src="/documents/images/os/thread/mode2.png" alt="sync" width="320" height="240">
  <span style="{{ site.img }}">picture [6] ULT</span>
</p>
<br>

kernel isn't aware of ULT.<br>
In the right case, if the process is blocked while the thread is in progress,<br>
all threads will die. Mismatch has occurred.<br>
There were many errors at the beginning of the thread.<br>

<span style="{{ site.code }}">KLT</span> :
**K**ernel **L**evel **T**hread. (kernel mode)<br>
multi-core is compatible. (parallel)<br>
Even if there is one process, multiple threads can be executed per core.<br>
null-fork/signal wait is very slow<br>
