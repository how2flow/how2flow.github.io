---
permalink: /documents/os/deadlock-and-starvation/
title: Deadlock and Starvation
excerpt: "Deadlock & Starvation documents"
toc: true
---

## Intro

The flow is as follows:
```
1. Ensured process isolation.
2. Ensured isolation, but there is a need to communicate between processes.
3. Provides semaphore.
4. Given exclusive rights to resources, there are cases where processes are sometimes blocked when they have resources.
5. Resources are limited.
6. As it builds up, the system doesn't work properly.
```

## Deadlock

The permanent blocking of a set of processes that either compete for system resources<br>
or communicate with each other.<br>

### Cases

It is a diagram illustrating a typical deadlock case.<br>

<p align="center">
  <img src="/documents/images/os/deadlock-and-starvation/case-simple.png" alt="case-simple" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [1] permanent deadlock</span>
</p>
<br>

But it doesn't really look as pretty as the picture above.<br>

<p align="center">
  <img src="/documents/images/os/deadlock-and-starvation/case-load.png" alt="case-load" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [2] The load: illustrate of deadlock</span>
</p>
<br>

It's a different situation this time.<br>

In the case of a car on the road, it can be solved by reversing.<br>
But the program can't reverse.<br><br>

Suppose there are processes <span style="{{ site.code }}">P</span> and <span style="{{ site.code }}">Q</span> .<br>
And look at the next picture.<br>
The part where the arrow progression is deflected to 90 degrees is the <span style="{{ site.code }}">context switching</span> .<br>

<p align="center">
  <img src="/documents/images/os/deadlock-and-starvation/case-example.png" alt="case-example" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [3] deadlock example</span>
</p>
<br>

If process goes to the red arrow path, deadlock does not occur.<br>
If process goes to the blue arrow path, deadlock will always occur.<br>
<span style="{{ site.code }}">Get</span> corresponds to <span style="{{ site.code }}">semWait</span> ,<br>
<span style="{{ site.code }}">Rel</span> corresponds to <span style="{{ site.code }}">semSignal</span> .<br>

### Resources categories

To avoid deadlocks, you need to know the nature of the resource.<br>
There are two main categories.<br>

#### reusable

It is a resource that continues to be used a time quantum.<br>
Like cpu, memory, i/O, etc ...<br>

#### consumable

created and destroyed resources.<br>
Representatively, there is communication.<br>
Like interrupt, message, etc ...<br>

### Condition

These are the conditions for the occurrence of deadlocks.<br>
The computer is deadlocked only when all conditions are met.<br>

```
1. mutual exclusion
Only one process may use a resource at a time.

2. hold-and-wait
A process may hold allocated resources while awaiting assignment of others.

3. no pre-emption
No resource can be forcibly removed from a process holding it.

4. circular wait
A closed chain of processes exists,
such that each process holds at least one resource needed by the next process in the chain
```

### Approach:prevention

It prevents deadlocks from occurring.<br>
The programmer make the program to prevent deadlocks from occurring. (static)<br>
Resolving at the Application Layer<br>

### Approach:avoidence

Running a program and the OS does not allocate resources in anticipation of a deadlock. <br>
Resolving at the OS Layer. (dynamic)<br>

Define two states for determination in os.<br>
<span style="{{ site.code }}">Safe-state</span> that is safe from deadlocks and <span style="{{ site.code }}">unsafe-state</span> that may cause deadlocks.<br>

It does not allocate resources to processes by comparing resources that have not yet been allocated,<br>
resources currently allocated to processes, and resources that are needed for each process, or Not initialize them in the case of new processes.<br>

### Approach:detection

It is to detect deadlocks.<br>
(Deadlock has occurred.)<br>
Kill and rerun processes involved in deadlocks.<br>
