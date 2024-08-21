---
permalink: /documents/gic-base/
title: GIC
excerpt: "Generic Interrupt Controller"
comments: false
toc: true
---

## GIC

GIC is 'Generic Interrupt Contoller'. 요약을 보려면, [GIC summary](/documents/gic-summary) 참조 하세요.<br>
GIC는 ARM에서 제공하는 ARM 전용 인터럽트 컨트롤러 이다.<br>

### Introduction

Interrupt Controller란,<br>
말 그대로 인터럽트를 제어하는 장치이다.<br>
외부 장치로부터 interrupt 신호를 수신하면,<br>
적절한 PE(Core)에 전달하는 역할을 한다.<br>
<br>

GIC를 구성하는 요소는 다음과 같다.
```
1. Distributor
2. Redistributor
3. CPU-INTERFACE
4. ITS (optional)
```
<br>

인터럽트 종류는 크게 4가지가 있다.<br>
```
1. SGIs
2. PPIs
3. SPIs
4. LPIs
```

## GIC components

크게<br>
<span style="{{ site.code }}">Distributor</span> , (<B>SPIs</B> route)<br>
<span style="{{ site.code }}">Redistributor</span> , (<B>SGIs</B>, <B>PPIs</B>, <B>LPIs</B> route)<br>
<span style="{{ site.code }}">CPU-INTERFACE</span> , (interface of CPU)<br>
<span style="{{ site.code }}">ITS</span> , (Translate <B>LPIs</B>)
가 있다.<br>

### Distributor

디스트리뷰터는 on-chip device나 external device에서<br>
요청하는 SPI를 분배하는 역할을 한다.<br>

### Redistributor

Re-디스트리뷰터는 SGIs 와 PPIs 핸들링 역할을 한다.<br>
LPIs의 메세지 기반의 인터럽트가 번역된 정보가 저장된<br>
메모리에 접근해서 정보를 읽고 분배하는 역할을 한다.<br>

### CPU-INTERFACE

cpu interface는 GIC가 core로 보내는 인터럽트 신호(IRQ/FIQ)를 Core에 전달하는 역할,<br>
Core의 Ack 신호를 GIC에 전달하는 역할을 한다.<br>
즉, GIC와 Core 사이의 소통을 위한 장치라고 보면 된다.<br>

### ITS (optional)

메세지 기반의 인터럽트의 메세지를 번역하는 역할을 한다.<br>
GICv3 부터 지원하지만, 필수 요소는 아니다.<br>
인터럽트 메세지를 DT / ITT / CT 를 참조하여 통해 번역한다.<br>

DT: Device Table (MSI -> Device ID)<br>
ITT: Interrupt Translation Table (MSI -> ICID)<br>
CT: Collection Table (ICID -> Translate Interrupt)<br>

<B>번역된 정보를 system memory에 적재한다.</B>
```
이 정보를 실제 커널 코드 분석을 통해 확인해야 한다.
```
<br>

## Interrupts

인터럽트는 하던 작업을 중단하고, 바로 처리해야 하는 것<br>
이라고 여기면 된다.<br>
인터럽트의 종류는 매우 많기 때문에 이를 관리해줄 컨트롤러가 필요하다.<br>
이를 인터럽트 컨트롤러라고 하며, GIC는 ARM에서 제공하는 인터럽트 컨트롤러 이다.<br>

인터럽트는 GIC와 Core의 신호에 따라 상태가 결정된다.<br>
'Inactive', 'Pending' 'Active' 'Active & Pending'<br>
인터럽트 type에 따라 인터럽트 상태 변화가 결정되며, 인터럽트 타입은 다음과 같다.
```
- Level sensitive
- Edge trigger
```
<br>

인터럽트 타입은 선택사항이지만, SGI와 LPI는 Edge trigger가 대부분이다.<br>
Level sensitive에 한해서, SPI는 <span style="{{ site.code }}">Active HIGH</span>, PPI는 <span style-"{{ site.code }}">Active LOW</span> 이다.<br>

<div style="text-align: center;">
  <img src="/documents/images/gic/gic_level_sens.png" alt="gic_level_sens" width="640" height="480"><br>
  <span style="{{ site.img }}">Picture [1] GIC level sensitive</span>
</div>
level sensitive는 pending 상태가 어떻게 정의되는지 잘 안보이지만,<br>
<span style="{{ site.code }}">peripheral -> gic</span> 신호가 HIGH라면 'pending 상태가 존재한다'라고 보면 된다.<br>
<br>

<div style="text-align: center;">
  <img src="/documents/images/gic/gic_edge.png" alt="gic_edge" width="640" height="480"><br>
  <span style="{{ site.img }}">Picture [2] GIC edge trigger</span>
</div>
edge trigger는 인터럽트가 처리 중일 때, 인터럽트 신호가 들어오면<br>
Active & Pending 상태라고 보면 된다.<br>
<br>

실제 인터럽트의 life-cycle에서 해당 상태에 따라 상태값을 저장하는 레지스터가 존재한다.<br>
이 부분은 GIC register set을 참조하면 된다.<br>
<br>

인터럽트의 종류에는 SGIs, PPIs, SPIs, LPIs가 있다.<br>
이 다음부터는 각 인터럽트 종류별 특성에 대해 서술한다.<br>

### SGIs

'<B>S</B>oftware <B>G</B>enerated <B>I</B>nterrupt' 이다.<br>
명령어를 통해 발생하는 인터럽트 이지만,<br>
GIC에서 IRQ 신호를 Core에 보내면서 익셉션을 유발하기 때문에,<br>
<U>하드웨어 인터럽트로 분류된다.</U><br>

SGIs는 인터럽트 번호 0 ~ 15번을 사용한다.<br>

### PPIs

'<B>P</B>rivate <B>P</B>rocessor <B>I</B>nterrupt' 이다.<br>
각 코어마다 연결되어 있는 Redistributor를 통해 분배된다.<br>
대표적으로 타이머가 있다.<br>

PPIs는 인터럽트 번호 16 ~ 31번을 사용한다.<br>

PPIs는 인터럽트 번호 16(16 + 0)번 부터 시작한다.<br>
16(16 + 0)이라고 표현한 이유는 SPIs 설명을 보면 알 수 있다.<br>

### SPIs

<B>S</B>hared <B>P</B>eripheral <B>I</B>nterrupts 이다.<br>
흔히 '인터럽트가 발생했다' 라고할 때 인터럽트가 SPI 이다.<br>
어느 코어에서든 처리가 가능한 인터럽트 이며, on-chip 이나 external device 대부분 이에 해당된다.<br>

SPIs는 인터럽트 번호 32 ~ 1019번을 사용한다.<br>

SPIs는 인터럽트 번호 32(32 + 0)번 부터 시작한다.<br>
32(32 + 0)이라고 표현한 이유는 예시를 보면 알 수 있다.<br>

ARMv8기반 칩 중 하나의 예(여기서 X칩 이라고 하겠다.)를 가져왔다.<br>
X칩의 인터럽트 테이블은 다음과 같다.
```
| 0 | EXTINT0
| 1 | EXTINT1
| 2 | EXTINT2
...
| 51 | IREQ_GPSBS
| 52 | IREQ_I2CM0
| 53 | IREQ_I2CM1
...
```
<br>

0번부터 시작해서 인터럽트 종류가 정해져 있다.<br>
실제 인터럽트 번호는 표에 나와있는 인터럽트 번호에 SPI의 오프셋인 32를 더해주어야 한다.(단, 칩 설계마다 다를 수 있음)<br>
예를 들어, I2CM0의 인터럽트 번호는 '(52 + 32) = <B>84</B>' 이다.
```
$ cat /proc/interrupts
        CPU0       CPU1       CPU2       CPU3       CPU4       CPU5       CPU6       CPU7
...

 63:      0          0          0          0          0          0          0          0     GICv3  84 Level     .....i2c0

...
```
이 84번 i2c 인터럽트를 실제 Trace32로 확인할 수 있다.<br>
<details>
  <summary>Trace32로 인터럽트 학인하는 예제</summary>
  <p>
    (이미지의 워터마크는 회사의 Trace32 라이센스를 사용했기 때문에 어쩔 수 없음)<br>
    인터럽트의 life-cycle은
    <pre><code>
    Generated - Distribute - Deliver - Pending - Activate - Priority Drop - Deactivation
    </code></pre>
    이다.<br><br>
 
    인터럽트 Active 상태일 때, 브레이크 포인트를 적절하게 설정하여<br>
    <span style="{{ site.code }}">ISACTIVR</span>, <span style="{{ site.code }}">ICACTIVER</span> 레지스터를 확인한다.
    <pre><code>
    Pending 상태를 확인하는 방법은 T32 사용으로는 난해하다.
    pending -> active 상태 넘어가는 기준은, core가 IAR 레지스터를 읽는 시점이며,
    읽는 순간 ACK 신호 발생 및, 인터럽트 번호 식별이 이루어 지기 때문이다.
    </code></pre>
    <br>

    먼저, X칩의 GIC 레지스터 주소는 다음과 같다.<br>
    <pre><code>
    0x17600000 | Main Cluster GIC
    </code></pre>
 
    예시로 드는 칩은 <span style="{{ site.code }}">GIC-600</span> 이 탑재되어 있어서, GIC 레지스터 셋은 <span style="{{ site.code }}">GIC-600</span> 을 참조한다.<br>
 
    Base address는 <span style="{{ site.code }}">0x17600000</span> 이고,<br>
    ISACTIVR의 offset은 <span style="{{ site.code }}">0x300</span> , ICACTIVER의 offset은 <span style="{{ site.code }}">0x380</span> 이다.<br>
 
    I2C 인터럽트 핸들러 진입 flow에서 break point를 걸고, 레지스터 dump를 했다.<br>
    다음 명령어를 통해 dump를 한다.
    <pre><code>
    B:: d.dump a:0x17600300 /sl
    </code></pre>
    <br>

    i2c0 사용하여 실제 84번 인터럽트가 active 되는지 확인한다.
    <pre><code>
    $ i2cdetect -y 0
    </code></pre>
    <br>

    <div style="text-align: center;">
      <img src="/documents/images/gic/i_activer.png" alt="gic_i_activer" width="640" height="480"><br>
      <span style="{{ site.img }}">picture [SPI - T32] Trace32 View I_ACTIVER</span>
    </div>
    <br>

    정확히 84번 인터럽트가 업데이트 되는 것을 확인할 수 있다.<br>

    이 상항에서 콘솔 로그는 다음과 같다.<br>
    <pre><code>
    [ERROR][I2C] transfer timeout, addr: a, result: 0
    [ERROR][I2C] transfer timeout, addr: b, result: 0
    [ERROR][I2C] transfer timeout, addr: c, result: 0
    ...
    </code></pre>

    ~~Trace32로 i2c 핸들러에 break point를 걸었기 때문에 인터럽트 처리 도중 멈추게 되었다.~~<br>
    정확히는, <span style="{{ site.code }}">ISPENDR</span> 도 함께 확인해야 한다.<br>
    왜냐하면, <span style="{{ site.code }}">edge-trigger type</span> 은 인터럽트 핸들러 진입 시, ISACTIVR는 set 되지 않는다.<br>
    당연하게도, <span style="{{ site.code }}">level-sensitive type</span> 과 인터럽트 상태 머신이 다르기 때문이다.<br>
    커널 디바이스트리로 인터럽트 타입을 변경하고 테스트를 하면 위의 사진과 다른 모습을 확인할 수 있다.<br>
    <br>

    한가지 더 고려할 부분이 있다. 위 이미지에서는 ICACTIVER도 함께 set 되어 있는데,<br>
    정확히는 clear 레지스터가 set이 되는 것을 조건으로 인터럽트 상태가 변화한다.<br>
    이는 ISPENDR / ICPENDR 역시 마찬가지 이다.<br>

    다만 함께 값이 변하는 것 처럼 보이는 것은, 예시 칩이 mirroring으로 설계 되어 있거나,<br>
    인터럽트 핸들러에서 clear 비트를 건들게 설계되어 있을 수 있다.<br>
    <br><br>

    T32 사용 시 주의사항으로,<br>
    커널 elf 올리고 Trace32를 사용할 경우에는<br>
    반드시 WDT(watchdog timer)를 disable 하고 사용해야 한다.<br>
    커널 입장에서는 칩이 비정상 멈춤 상태로 인식하고 WDT로 강제로 Reset 하기 때문이다.<br>
  </p>
</details>


### LPIs

2가지 보안상태를 지원하는 환경에서,<br>
<span style="{{ site.code }}">LPIs</span> 는 항상 Non-secure Gruop 1 이다.<br>
<br>

2가지 보안상태란,(GICD_CTLR.DS=0)<br>
첫번째는, Group 0 와 Gruop 1으로 나누는 것,<br>
두번째는, Secure Group 1과 Non-secure Group 1 으로 나누기 때문에<br>
2가지 보안상태를 지원하고, 2가지 보안상태라고 한다.<br>
1가지 보안상태라면(GICD_CTLR.DS=0), Group 1 이다.<br>
<br>

LPIs는 무조건 <span style="{{ site.code }}">Edge-Trigger</span> 방식을 지원한다.<br>
각 인터럽트의 pending 정보는 <U>레지스터가 아닌 메모리</U> 내의<br>
테이블에 저장되며 이 테이블은 <U>Redistributor에 저장된 레지스터</U>에 의해 참조된다.<br>
메모리에 인터럽트의 정보를 저장하는 이유는 LPIs는 다른 종류의 인터럽트들 보다<br>
훨씬 개수가 많기 때문이다.<br>
<br>

위 내용은 <span style="{{ site.code }}">kernel</span> 이나 <span style="{{ site.code }}">tf-a</span> 코드를 통해 반드시 확인해야 한다.<br>
<br>

## Power management

## Programmer's model
