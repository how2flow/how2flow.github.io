---
permalink: /documents/os-overview/
title: Operating System
toc: true
---

<a href="{{ site.baseurl }}/documents/os-index/">Return to Index</a><br>

## Chap1. Overview

This post is an overview. Just think about it like "Most of them have this and that" and move on.<br>
The concepts shown here will be covered in detail in a later posting.<br>
<span style="{{ site.ko }}"> 이 게시물은 개요입니다. "대부분 이것저것 가지고 있다"는 식으로 생각하고 넘어가세요. 여기에 표시된 개념은 나중에 게시물에서 자세히 다룰 것입니다.</span><br>

---

Anyone who has used a computer has heard the word OS.<br>
<span style="{{ site.ko }}">컴퓨터를 사용해 본 사람이라면 OS라는 단어를 들어본 적 있을 겁니다.</span><br>
<br>
Windows, macOS, Linux, etc., which we've heard about once in our lives, are all operating systems.
<span style="{{ site.ko }}">우리가 살면서 한번 쯤 들어봤을 windows, macOS, Linux 등 모두 Operating system입니다.</span><br>
<br>
Why do I need an OS? It is necessary for resource management.<br>
ls I will talk about OS in the next, resources(cpu, memory, etc ...) are limited and very important.<br>
<span style="{{ site.ko }}">OS 가 필요한 이유는 무엇입니까? 제한적인 리소스 관리(cpu, memory, etc ...)를 위해 필요합니다. 리소스는 다음에 이어서 설명하겠습니다.</span><br>
First, I will briefly explain the **computer system** and **OS**.<br>
<span style="{{ site.ko }}">먼저 컴퓨터 시스템과 운영체제에 대해 간단히 설명하겠습니다.</span><br>

### Computer System

Before we get into the OS, let's take a look at the CPU and resister that are at the heart of the computer.
<span style="{{ site.ko }}">OS에 대해 제대로 알아보기 전에, 컴퓨터 시스템에서 가장 핵심이 되는 CPU와 레지스터에 대해 알아보겠습니다.</span><br>
<br>
The CPU consists largely of ALU, registers, and I/Operipherals. (Three Elements of CPU)<br>
<span style="{{ site.ko }}">CPU는 크게 ALU, 레지스터, I/O peripherals로 구성되어있습니다. (CPU의 3요소)</span><br>
<br>
Resistors are architecturally distinct, but are mostly divided into regular and special registers.<br>
<span style="{{ site.ko }}">그리고 레즈스터는 아키텍쳐마다 다르지만 대부분 일반 레지스터와 특수 레지스터로 구분됩니다.</span><br>
<br>
Special registers include PC (Program Counter), IR (Instruction Register), MAR (Memory Address Register), and MBR (Memory Buffer Register).<br>
<span style="{{ site.ko }}">특수 레지스터로는 PC(프로그램 카운터), IR(명령 레지스터), MAR(메모리 주소 레지스터), MBR(메모리 버퍼 레지스터) 등이 있습니다.</span><br>
<br>

**- Memory Hierarchy**

Memory has layers.
<span style="{{ site.ko }}">메모리는 계층을 가지고 있다.</span><br>
```
                        (fast)
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
<span style="{{ site.ko }}">CPU에 가까울 수록 빠르고 HDD (요즘은 SSD나 NVME도 병렬로 많이 사용한다.) 에 가까울 수록 느리고 용량이 큽니다.</span><br>

**- Computer Layout**

This is a summary of the CPU and Computer.
<span style="{{ site.ko }}">CPU와 컴퓨터 구조를 단순하게 표현한 그림입니다.</span><br>
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
<span style="{{ site.ko }}">컴퓨터는 크게 CPU, 메인 메모리, I/O 주변기기로 구성되며, CPU는 ALU, 메모리 영역 레지스터, 제어 장치로 구성된다.</span><br>

### Operating System

OS is an interface between application and hardware.<br>
<span style="{{ site.ko }}">OS는 어플리케이션과 하드웨어사이에 존재하는 하나의 인터페이스 입니다.</span><br>

If you look at the overall structure briefly,
<span style="{{ site.ko }}">전체적인 구조를 간략히 살펴보면, </span><br>

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
<span style="{{ site.ko }}">위와 같이 표현할 수 있습니다.</span><br>

#### Upgrade steps of OS

I'll describe the concept of OS with the development steps of OS.<br>
<span style="{{ site.ko }}">OS의 개념을 OS의 발전단계(과정)와 함께 설명하겠습니다.</span><br>

| step | Index |
|:---:|:---:|
| step 5| multi physics (multi core)|
| step 4| time-sharing|
| step 3| multi programming batch system|
| step 2| simple batch system|
| step 1| serial process|

**1. serial process**

It's the beginning of the process, It is not Operating System.<br>
<span style="{{ site.ko }}">프로세스의 시초격이지만, 이 개념은 OS라 할 수 없습니다.</span><br>

**2. simple batch system**

It is a method of sequentially processing programs over time.<br>
<span style="{{ site.ko }}">시간에 따라 프로그램을 순차적으로 진행하는 방식입니다.</span><br>
e.g.
```

+--- monitor ---+                 +--- monitor ---+                 +--- monitor ---+
|               |                 |               |                 |               |
|               | => program A => |               | => program B => |               | => ...
|               |                 |               |                 |               |
+---------------+                 +---------------+                 +---------------+

```

These are the conditions for the desirable behavior shown at this stage.<br>
<span style="{{ site.ko }}">이 단계에서 나타난 바람직한 동작을 위한 조건들 입니다.</span><br>
- monitor memory protection.
- Timer.
- privileged Instruction. (user mode / kernel mode)
- Interrupt.

**3. multi programming batch system.**

It's to keep various programs running nonstop.<br>
How multiple user programs are handled as if they were running simultaneously.<br>
The concept of <span style="{{ site.important }}">concurrency</span> emerges.<br>
<span style="{{ site.important }}">Parallel processes of multi core and concurrency are different!!!</span><br>
<span style="{{ site.ko }}">여러 프로그램들을 쉬지 않고 동작시키는 것입니다.</span><br>
<span style="{{ site.ko }}">마치 여러 개의 사용자 프로그램이 동시에 실행되는 것처럼 처리하는 방식입니다.</span><br>
<span style="{{ site.ko }}">여기서</span><span style="{{ site.important }}">concurrency</span><span style="{{ site.ko }}">의 개념이 등장합니다.</span><br>
<span style="{{ site.important }}">멀티 코어의 병렬 프로세스와 동시성은 다릅니다!!!</span><br>

e.g.
```
case 1. [A] ...wait... [B] ...wait... [C] ...
case 2. .. [D] ...wait... [E] ...wait... [F] ...

...

======> [A][D] ....... [B][E] ....... [C][F] ...

```

**4. time-sharing**

Improved user responsiveness over multi-deployment approaches. In general<br>
Each program sets its own time and runs only that amount of time and moves on to the next program.<br>
Typically, there is a round-robin method.<br>
<span style="{{ site.ko }}">multy batch system 보다 사용자 응답성이 발전된 방식입니다.</span><br>
<span style="{{ site.ko }}">각 프로그램은 자체 시간을 설정하고 해당 시간만 실행하고 다음 프로그램으로 넘어갑니다.</span><br>
<span style="{{ site.ko }}">일반적으로 라운드 로빈 방식이 있습니다.</span><br>

**5. multi physics (multi core)**

It literally means multiple cores.<br>
OS documentation is based on <span sytle="{{ site.important }}">single core.</span><br>
<span style="{{ site.ko }}">말 그대로 코어가 여러개인 것을 뜻합니다.</span><br>
<span style="{{ site.ko }}">OS문서에서는 <span sytle="{{ site.important }}">싱글코어</span>를 기준으로 서술합니다.</span><br>

<a href="{{ site.baseurl }}/documents/os-process/">Next =></a><br>
