---
permalink: /documents/os/concurrency/
title: Concurrency
excerpt: "How to ensure Operating System Concurrency?"
toc: true
---

## Concurrency

Dispatchers ensure process isolation.<br>
Allocate time to run the thread.<br>

It's 'running' because the thread started and didn't end.<br>
There is one core, but multiple threads are 'running', ensuring concurrency.<br>

Unlike threads, memory cannot be divided over time.<br>
Memory is limited and allocated by dividing it into spaces.<br>

The OS swindles the process that it monopolizes all resources, but in reality,<br>
resources are limited and used by everyone.<br>
Traffic arrangement is necessary when trying to take limited resources with each other.<br>

Limited resources should be mutual exclusion. And you have to synchronize what you use.<br>
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
  <img src="/assets/images/documents/os/concurrency/atomic1.png" alt="atomic1" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [1] Atomic Operation 1</span>
</p>
<br>

When two processes are completed, the result should be 100.<br>
However, if context switching occurs as follows, the result will be different.<br>

<p align="center">
  <img src="/assets/images/documents/os/concurrency/atomic2.png" alt="atomic2" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [2] Atomic Operation 2</span>
</p>
<br>

So This case needs <span style="{{ site.code }}">mutual exclusion</span> .<br>
The exclusion use of <span style="{{ site.code }}">a</span> should be guaranteed.<br>
<br>

Here's an example.<br>

<p align="center">
  <img src="/assets/images/documents/os/concurrency/mutualexclusion.png" alt="mutual_ex" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [3] Mutual Exclusion</span>
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

The problem with ensuring this mutual exclusion is that there are different types and<br>
numbers of shared resources, processes, and the shared resources required by each process.<br>
<br>

I’ll give you an example.<br>

Process 1 requires resource 1 and resource 2, and process 2 also requires resource 1 and resource 2,<br>
and when resource 1 is allocated to process 1 and resource 2 is allocated to process 2, its own resource has other resources.<br>
Process 1 and process 2 are not executed forever.<br>

<p align="center">
  <img src="/assets/images/documents/os/concurrency/deadlock.png" alt="deadlock" width="640" height="480"><br>
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
It is said to support <span style="{{ site.code }}">mutual exclusion</span> .<br>
When <span style="{{ site.code }}">mutual exclusion</span> is guaranteed, a <span style="{{ site.code }}">deadlock</span> occurs that is waiting for the other party's work to be completed.<br>
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

## Mutual Exclusion Implementing

### Hardware Support

Atomic operations shall be implemented in critical sections.<br>

#### blocking interrupt
```
Turn off interrupt before entering critical section and turn on interrupt when critical section is finished.
```
However, processes that are not related to shared resources are all put on waiting.<br>
In addition, even if it is a process that uses shared resources, threads before using resources are also in a waiting state.<br>
It is very inefficient.<br>
When the interrupt is turned off, it does not work on the multiprocessor.<br>

#### Compare & Swap Instruction

also called a <span style="{{ site.code }}">compare and exchange instruction</span> .

first, make a flag and make two simple code.<br>
Assume that it enters the foo function and is called at the end of the bar function.<br>
```
/* Program 1 */
 1 int flag;
 2
 3 foo() { // enter
 4 	while (flag == 0);
 5 		flag = 0;
 6 }
 7
 8 bar() { // exit
 9 	flag = 1;
10 }
```
```
/* Program 2 */
 1 int flag;
 2
 3 foo() { // enter
 4 	while (flag == 0);
 5 		flag = 0;
 6 }
 7
 8 bar() { // exit
 9 	flag = 1;
10 }
```

It looks like it's been implemented,<br>
If there is an interruption at line 4, and context switching occurs,<br>
Both programs will enter the critical section.<br>

You can also blocking interrupt line 4 up and down,<br>
but it has become a little more efficient from the initial description,<br>
and the problem remains the same.<br>

So there is a command that supports reading and writing flag values at once.<br>
That's what a <span style="{{ site.code }}">compare and swap is</span> .
```
 1  /* program mutualexclusion */
 2 const int n = /* number of processes */;
 3 int flag; /* 0: allow 1: disallow */
 4
 5 void foo(int i) {
 6 	while (true) {
 7 		while (compare_and_swap(&flag, 0, 1) == 1); /* read & write at once */
 8 		/* critical section */
 9 		flag = 0;
10 		/* remainder */
11 	}
12 }

void main() {
	flag = 0;
	parbegin (P(1), P(2), ... , P(n));
}
```

another way is <span style="{{ site.code }}">exchange</span> .
```
 1 /* program mutualexclusion */
 2 const int n = /* number of processes */;
 3 int flag;
 4 
 5 void bar(int i) {
 6 	while (true) {
 7 		int keyi = 1;
 8 		do exchange (&keyi, &flag)
 9 		while (keyi != 0);
10  		/* critical section */
11  		flag = 0;
12  		/* remainder */
13  	}
14 }
15  
16 void main() {
17 	flag = 0;
18 	parbegin (P(1), P(2), ... , P(n));
19 }
```

##### advantages & disadvantages

advantages
```
- Applicable to any number of processes on either a uni or multi.
- Simple and easy to verfy.
- It can be used to support mulitple critical sections by its own variable.
```

disadvantages
```
- Busy waiting
- Starvation
- Deadlock
```

### Ways of Mutual Exclusion

| Name | Info |
|:---|:---|
| Semaphore | general |
| Binary Semaphore | Use Binary Semapore var |
| Mutex | Lock & unlock only in one process. |
| Condition Variable | support by program language |
| Monitor | support by program language |
| Event Flags |
| Maliboxes/Messages | synchronize by communication |
| Spinlocks | use in multiprocessor |

## Semaphore

It is variable. type is integer.<br>
The value of the semaphore is the number of accessible processes.<br>
It has some operations.
```
- Initialize to a nonnegative integer value
- Use semWait # decrement
- Use semSignal # increment
```

### example of Semaphore

```
 1 struct semaphore {
 2 	int count;
 3 	queueType queue; // FIFO
 4 };
 5
 6 void semWait(semaphore s) {
 7 	s.count--;
 8 	if (s.count < 0) {
 9 		/* place this process in s.queue */
10 		/* block this process */
11 	}
12 }
13
14 void semSignal(semaphore s) {
15 	s.count++;
16 	if (s.count <= 0) {
17 		/* remove a process from s.queue */
18 		/* place process on ready list */
19 	}
20 }
```
The fact that Semaphore's value has become negative means that it is impossible to enter.<br>
The negative value of the semaphore is the number of blocked because they did not enter.<br>

When a semaphore signal is called and one process exits the critical section,<br>
The first (or higher priority) process that is blocked is caught in the ready queue.<br>

### example of Binary Semaphore

```
 1 struct binary_semaphore {
 2 	int value = 0;
 3 	queueType queue;
 4 };
 5
 6 void semWait(binary_semaphore s) {
 7 	if (s.value == 1)
 8 		s.value = 0;
 9 	else {
10 		/* place this process in s.queue */
11 		/* block this process */
12 	}
13 }
14
15 void semSignal(binary_semaphore s) {
16 	if (s.queue is empty())
17 		s.value = 1;
18 	else {
19 		/* remove a process from s.queue */
20 		/* place process on ready list */
21 	}
22 }
```

### examples of Semaphore Machanism

It is an image representing a semaphore operation.<br>
It can be seen that the state of the process changes<br>
while ensuring mutual exclusion as shared resources are used and access to critical sections.<br>

<p align="center">
  <img src="/assets/images/documents/os/concurrency/semaphoreq1.png" alt="semaphoreq1" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [5-1] Semaphore Queue 1</span>
</p>
<p align="center">
  <img src="/assets/images/documents/os/concurrency/semaphoreq2.png" alt="semaphoreq2" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [5-2] Semaphore Queue 2</span>
</p>
<br>

The semaphore variable value is equal to the number of processes waiting in the blocked queue.<br>
It may not be a unicore system, or it may vary depending on the situation.<br>

### semaphore vs blocking interrupt

I assume that there is a process that performs +1 and -1 on the shared resource,<br>
and the result is a program that wants the same value as the first one.<br>
Let's compare inter-exclusion with blocking interrupt and semaphore.<br>

<p align="center">
  <img src="/assets/images/documents/os/concurrency/sharedi1.png" alt="sharedi1" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [6-1] No ensure mutual exclsion</span>
</p>
<br>
<p align="center">
  <img src="/assets/images/documents/os/concurrency/sharedi2.png" alt="sharedi2" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [6-2] Blocking Interrupt</span>
</p>
<br>
<p align="center">
  <img src="/assets/images/documents/os/concurrency/sharedi3.png" alt="sharedi3" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [6-3] Using Semaphore </span>
</p>
<br>

Blocking interrupts requires waiting for processes that are not related to shared resources when a process approaches a critical section.<br>
It's inefficient. Above all, it cannot be used in multi-core systems.<br>

Semapore does not block interruptions,<br>
so it is free to switch processes and can be applied to multi-core systems.<br>

## ETC

A system example that guarantees automatic-exclusive and synchronization like semaphores.<br>

### Monitors

Programming language constrcut that provides equivalent functionality<br>
to that of semaphores and is easier to control.<br>

#### Program languages

Pascal, Pascal-Plus, Modula-2, Modula-3, and Java<br>

#### Characteristics

Local data vars are accessible only by the Monitor's procedures and not by any external procedure.<br>
Process enters monitor by invoking one of its procedures(functions).<br>
Only one process may be executing in the monitor at a time.<br>

#### synchronization

Achieved by the use of <span style="{{ site.code }}">Condition variables</span> that are combined within the monitor and accessible only within the monitor.<br>
<span style="{{ site.code}}">Condition variables</span> are operated on by two functions:
```
cwait(c): suspend execution of the calling process on condition c.
csignal(c): resume execution of some process blocked after a cwait on the same condition.
```

### Message Passing

Direct communication between processes with <span style="{{ site.code }}">IPC</span> .<br>

<p align="center">
  <img src="/assets/images/documents/os/concurrency/ipc.png" alt="ipc" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [7] Message Passing </span>
</p>
<br>

works with <span style="{{ site.code }}">Distributed systems</span> and <span style="{{ site.code }}">shared memory multiprocessor</span> and <span style="{{ site.code }}">Uniprocessor systems</span> .<br>

#### format

The actual function is normally provided in the form of a pair of primitives:
```
send(destination, message)
receive(source, message)
```

<p align="center">
  <img src="/assets/images/documents/os/concurrency/message-format.png" alt="message-format" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [8] Message Format</span>
</p>
<br>

Message Passing are exchanged in TLV format after the source and destination addresses are determined.<br>

<p align="center">
  <img src="/assets/images/documents/os/concurrency/TLV.png" alt="message-format" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [9] TLV</span>
</p>
<br>

There are fixed and variable message lengths.<br>
It is usually used variable types because processes don't know the length of messages that are actually exchanged.<br>
<span style="{{ site.code }}">CRC</span> is optional.<br>

#### addressing

There are two types of addressing: direct and indirect.<br>

<p align="center">
  <img src="/assets/images/documents/os/concurrency/direct-addressing.png" alt="direct-addressing" width="320" height="240">
  <img src="/assets/images/documents/os/concurrency/indirect-addressing.png" alt="indirect-addressing" width="320" height="240"><br>
  <span style="{{ site.img }}">picture [10] addressing</span>
</p>
<br>

The direct format is a method of designating and using a send/receive process with a <span style="{{ site.code }}">PID</span>, etc.<br>
The redirect format is used in <span style="{{ site.code }}">broadcasts</span> that are sprayed to an unspecified number of people, or in systems that send messages and do not require feedback, etc.<br>

#### mailbox-code

This is an example of mutualexclusion with mailbox.
```
 1 const int n = /* number of processes */
 2 void P(int i) {
 3 	message msg;
 4 	while (true) {
 5 		receive (box, msg);
 6 		/* critical section */
 7 		send (box, msg);
 8 		/* remainder */
 9 	}
10 void main() {
11 	create_mailbox (box);
12 	send(box, null);
13 	parbegin (P(1), P(2), P(3), ..., P(n));
14 }
}
```


