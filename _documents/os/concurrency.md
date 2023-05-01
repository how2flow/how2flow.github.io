---
permalink: /documents/os/concurrency/
title: Concurrency
toc: true
---

## Chap 4. Concurrency

Dispatchers ensure process isolation.<br>
Allocate time to run the thread.<br>

It's 'running' because the thread started and didn't end.<br>
There is one core, but multiple threads are 'running', ensuring concurrency.<br>

Unlike threads, memory cannot be divided over time.<br>
Memory is limited and allocated by dividing it into spaces.<br>

The OS swindles the process that it monopolizes all resources, but in reality,<br>
resources are limited and used by everyone.<br>
Traffic arrangement is necessary when trying to take limited resources with each other.<br>

Limited resources should be mutually exclusive. And you have to synchronize what you use.<br>
It is necessary to ensure concurrency.<br>

### multiple process

OS design is concerned with the management of processes and threads:

```
- Multi programming
- Multi processing (multi core)
- Distributed Processing (Multiple computers are connected by a network, etc)
  `clustering
```

### Terms of Concurrency

<span style="{{ site.code }}">atomic operation</span> : <br>
Operations that do not lose execution permission in the middle when certain conditions are present.<br>

<span style="{{ site.code }}">critical section</span> : <br>
The protected sections.<br>

<span style="{{ site.code }}">race condition</span> : <br>
The state of trying to use resources with each other.<br>

<span style="{{ site.code }}">mutual exclusion</span> : <br>
Using resources entirely alone.<br>

<span style="{{ site.code }}">deadlock</span> : <br>
The status of waiting for the required resources. And if that resource remains occupied by another process.<br>
A state of being unable to do anything after all.<br>

<span style="{{ site.code }}">livelock</span> : <br>
In a state such as a deadlock, But The program is in operation(?)<br>

<span style="{{ site.code }}">starvation</span> : <br>
A state in which certain resources are required but are not assigned.<br>
<br>

Suppose you have two programs.<br>
It is written by C.<br>
```

static int a = 100;

int Program1() {
	a = a + 10;
	return a
}

int Program2() {
	a = a - 10;
	return a
}

...
	Program1();
	Program2();
...
```
It is assumed that concurrency is guaranteed by time quantum of time.<br>
Of course, C Program is eventually converted into assembly instructions.<br>

<p align="center">
  <img src="/documents/images/os/concurrency/atomic1.png" alt="atomic1" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [1] Atomic Operation 1</span>
</p>
<br>

When two processes are completed, the result should be 100.<br>
However, if context switching occurs as follows, the result will be different.<br>

<p align="center">
  <img src="/documents/images/os/concurrency/atomic2.png" alt="atomic2" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [2] Atomic Operation 2</span>
</p>
<br>

So This case needs <span style="{{ site.code }}">mutual exclusion</span> .<br>
The exclusive use of <span style="{{ site.code }}">a</span> should be guaranteed.<br>
<br>

Here's an example.<br>

<p align="center">
  <img src="/documents/images/os/concurrency/mutualexclusive.png" alt="mutual_ex" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [3] Mutual Exclusive</span>
</p>
<br>

There is only one toilet in the park, so it is assumed that it is for one person only.<br>
But if multiple people have to go to the bathroom, what should we do?<br>

Here, people correspond to the process, and the toilet bowl in the bathroom corresponds to a shared resource.<br>
What if someone opens the door when you poop?<br>

Like you're not supposed to go into the bathroom when someone's using it,<br>
When a process uses shared resources, it should not compete with shared resources.<br>

Like hanging a sign while using the toilet and putting it down when you're done using it,<br>
Raise the flag when the process uses shared resources and lower the flag when it is finished.<br>
The flag is usually used as a variable, and an example is a 'semaphore'<br>

The problem with ensuring this mutual exclusivity is that there are different types and<br>
numbers of shared resources, processes, and the shared resources required by each process.<br>
<br>

I’ll give you an example.<br>

Process 1 requires resource 1 and resource 2, and process 2 also requires resource 1 and resource 2,<br>
and when resource 1 is allocated to process 1 and resource 2 is allocated to process 2, its own resource has other resources.<br>
Process 1 and process 2 are not executed forever.<br>

<p align="center">
  <img src="/documents/images/os/concurrency/deadlock.png" alt="deadlock" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [4] Dead Lock</span>
</p>
<br>

Such a difficult situation can come.<br>
A state in which processes are just waiting for each other and nothing is executed is called a <span style="{{ site.code }}">deadlock</span> .<br>
If you prioritize a process separately to resolve the problem, some processes may never be allocated resources.<br>
It's called <span style="{{ site.code }}">starvation</span> .<br>

#### summary
The state in which there are shared resources and various processes want to use each other is called <span style="{{ site.code }}">race condition</span> .<br>
In order to solve the <span style="{{ site.code }}">race condition</span> , the <span style="{{ site.code }}">critical section</span> should be made into an <span style="{{ site.code }}">atomic operation</span> .<br>
It is said to support <span style="{{ site.code }}">mutual exclusive</span> .<br>
When <span style="{{ site.code }}">mutual exclusivity</span> is guaranteed, a <span style="{{ site.code }}">deadlock</span> occurs that is waiting for the other party's work to be completed.<br>
If you give priority to resolve deadlocks, A process that continues to fall behind priorities and is unable to allocate resources is called <span style="{{ site.code }}">starvation</span> .<br>

### Principles of Concurrency

Interleaving and overlapping.<br>
Uniprocessor - the relative speed of execution of processes can't be predicted.
```
Uni - processor : (Interleaving only)

	| P 1 | --- | P 1 | --- |
	| --- | P 2 | --- | --- | ...
	| --- | --- | --- | P 3 |



Multi - processor : (Interleaving + Overlapping)
	| P 1 | --- | P 1 | --- |
	| --- | P 2 | --- | --- | ...
	| --- | --- | --- | P 3 |
	`-----`-----`-----`-----`  ...
	 `-----`-----`-----`-----`
	  `-----`-----`-----`-----`  ...
	   `-----`-----`-----`-----`
```
<span style="{{ site.code }}">overlapping</span> :<br>
Since there are multiple cores, different processes are physically executed at the same time.<br>

### Difficulties of Concurrency

Considerations when ensuring concurrency.

```
- Sharing of global resources.
- Difficult for the OS to manage the allocation of resources optimally.
- Difficult to locate programming errors as result are not deterministic and reproducible.
- It may be a problem of its own, but it is also related to competition with other processes.
```

### Race Condition

multiple process or threads read and write data items.<br>
The final result depends on the order of execution.<br>

Resource allocation should not be timed.<br>
If you adjust the execution speed, it should depend on the user's computer and how many processes there are.<br>
So it should never be solved by timing.<br>

### Process Interaction

| Degree of Awareness | Relationship | Potential Problems |
|:---|:---|:---|
| Unaware of each other | Competition | - Mutual exclusion<br>- Deadlock<br>- Starvation<br> |
| Indirect aware<br>(shared object) | Competition by sharing | - Mutual exclusion<br>- Deadlock<br>- Starvation<br>- Data coherence |
| Directly aware | Cooperation by communication | - Deadlock<br>- Starvation |

### Requirements for Mutual Exclusion

```
- Must be enforced.
- A process that halts must do so without interfering with other processes
- No deadlock
- No starvation
- A process must not be denied access to a critical section when there is no toher process using it.
- No assumptions are made about relative process speeds or number of prcesses.
- A process remains inside its critical section for a finite time only.
```
