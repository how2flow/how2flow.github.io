---
permalink: /legacy/linux_irq_manage
title: "[Linux] IRQ management" 
excerpt: "리눅스 커널의 인터럽트 관리"
header:
  teaser: /assets/images/legacy/note.jpg
categories:
  - Linux
tags:
  - interrupt
toc: true
---

## 소개

리눅스 커널의 인터럽트 관리는 어떻게 되고 있는지에 대한 설명(ver = 5.10) 입니다.<br>

```
 ,-->  hwirq
 |          |
 `--------> `<irq_desc>
  `           |       |
   `----------------> `<irq_data>
    `         |         |       |
     `------------------------> `<irq_chip>
              |         |                 .irq_startup
              .handler  |                 .irq_shutdown
                        .hwirq <--> irq   .irq_enable
                        .domain           .irq_disable
                        .parent_data      .irq_ack
                                          .irq_mask
                                          .irq_unmask
                                          .irq_eoi
                                          .irq_set_affinity
                                          ...
                                          .parent_device
```

## hwirq

hwirq는 실제 하드웨어가 가지고 있는 인터럽트 고유 번호입니다.<br>
리눅스 커널은 이 번호를 가상 인터럽트 번호로 매핑하여 관리합니다.<br>
개발자는 <span style="{{ site.code }}">device-tree</span> 에 하드웨어 인터럽트 번호를 작성합니다.
```
	label: device@10000000 {
		status = "okay";
		compatible = "samsung,device";
		interrupts = <GIC_SPI 38 IRQ_TYPE_LEVEL_HIGH>;
		...
	}
```
<br>

38번은 하드웨어 인터럽트 번호이며, SoC나 디바이스 데이터 시트에 해당 정보를 확인할 수 있습니다.<br>
리눅스 커널에서는 <span style="{{ site.code }}">hwirq</span> 으로 관리 됩니다.<br>


## IRQ 정보 관리

리눅스 커널은 IRQ 정보를 다양한 오브젝트로 관리합니다.<br>
인터럽트에 대한 정보는 <span style="{{ site.code }}">irq_chip</span> , <span style="{{ site.code }}">irq_desc</span> , <span style="{{ site.code }}">irq_data</span> 등이 있습니다.<br>

### irq_desc

<span style="{{ site.code }}">irq_desc</span> 는 IRQ 라인의 최상위 디스크립터 입니다.<br>
해당 IRQ의 전체 상태를 저장합니다.<br>
그리고 해당 IRQ에 연결된 핸들러 정보를 매핑하고 있습니다.<br>

구조체의 기본형태는 다음과 같습니다. (include/linux/irqdesc.h)
```
struct irq_desc {
    struct irq_common_data  irq_common_data;
    struct irq_data     irq_data;
    unsigned int __percpu   *kstat_irqs;
    irq_flow_handler_t  handle_irq;
    struct irqaction    *action;    /* IRQ action list */

    ...

    struct mutex        request_mutex;
    int         parent_irq;
    struct module       *owner;
    const char      *name;
} ____cacheline_internodealigned_in_smp;
```
<br>

<span style="{{ site.code }}">irq_data</span> 을 포함하고 있기 때문에, <span style="{{ site.code }}">irq_desc</span> 를 알고 있으면 <span style="{{ site.code }}">irq_data</span> 를 불러올 수 있습니다.<br>
역으로도 <span style="{{ site.code }}">container_of</span> 매크로를 활용하면 가능합니다.<br>
커널에 구현되어 있습니다.<br>

### irq_data

irq_desc 내부(또는 계층 구조상)에서, 실제 하드웨어 IRQ 번호나, IRQ를 제어하기 위한 정보(irq_chip, 하드웨어 관련 플래그 등)를 담는 구조체 입니다.<br>
하드웨어 레벨의 IRQ 번호(hwirq), 트리 구조(부모/자식 irq_data), IRQ line의 옵션(마스크, 트리거 방식 등) 정보를 저장합니다.<br>
irq_chip을 통해 제공되는 메서드를 호출할 때 필요한, SoC나 아키텍처 의존적인 데이터를 저장할 수도 있습니다.<br>

구조체의 기본형태는 다음과 같습니다. (include/linux/irq.h)
```
struct irq_data {
    u32         mask;
    unsigned int        irq;
    unsigned long       hwirq;
    struct irq_common_data  *common;
    struct irq_chip     *chip;
    struct irq_domain   *domain;
#ifdef  CONFIG_IRQ_DOMAIN_HIERARCHY
    struct irq_data     *parent_data;
#endif
    void            *chip_data;
};
```
<br>

<span style="{{ site.code }}">irq_chip</span> 을 포함하고 있기 때문에, <span style="{{ site.code }}">irq_data</span> 를 알고 있으면 <span style="{{ site.code }}">irq_chip</span> 를 불러올 수 있습니다.<br>
역으로도 <span style="{{ site.code }}">container_of</span> 매크로를 활용하면 가능합니다.<br>
커널에 구현되어 있습니다.<br>

### irq_chip

실제 하드웨어 인터럽트를 활성화/비활성화(mask/unmask), ack, eoi(End Of Interrupt) 등의 동작을 구현하는 함수 포인터 테이블입니다.<br>
해당 IRQ 라인(또는 그룹)에 대한 제어 함수를 제공합니다.<br>
<span style="{{ site.code }}">irq_chip->irq_ack</span> , <span style="{{ site.code }}">irq_chip->irq_mask</span>, <span style="{{ site.code }}">irq_chip->irq_unmask</span>, <span style="{{ site.code }}">irq_chip->irq_eoi</span> 등<br>
보통 ARM GIC(Generic Interrupt Controller), GPIO 기반의 interrupt controller, PCI interrupt 등 하드웨어마다 다른 인터럽트 제어 로직을 각각 구현해 놓습니다.<br>

구조체의 기본형태는 다음과 같습니다. (include/linux/irq.h)
```
struct irq_chip {
    struct device   *parent_device;
    const char  *name;
    unsigned int    (*irq_startup)(struct irq_data *data);

    ...

    void        (*irq_ack)(struct irq_data *data);
    void        (*irq_mask)(struct irq_data *data);
    void        (*irq_mask_ack)(struct irq_data *data);
    void        (*irq_unmask)(struct irq_data *data);
    void        (*irq_eoi)(struct irq_data *data);

    int     (*irq_set_affinity)(struct irq_data *data, const struct cpumask *dest, bool force);

    ...

    unsigned long   flags;
};

```
<br>

## IRQ handler

리눅스 커널에서 인터럽트 핸들러 함수는 <span style="{{ site.code }}">irqreturn_t</span> 타입을 갖습니다.<br>
각 IRQ의 핸들러는 <span style="{{ site.code }}">irq_desc</span> 의 <span style="{{ site.code }}">irqaction</span> 에서 관리됩니다.<br>
(include/linux/interrupt.h)
```
typedef irqreturn_t (*irq_handler_t)(int, void *);

...

struct irqaction {
    irq_handler_t       handler;
    void            *dev_id;
    void __percpu       *percpu_dev_id;
    struct irqaction    *next;
    irq_handler_t       thread_fn;

    ...

    unsigned int        irq;
    unsigned int        flags;
    unsigned long       thread_flags;
    unsigned long       thread_mask;
    const char      *name;
    struct proc_dir_entry   *dir;
} ____cacheline_internodealigned_in_smp;

```
<br>

### irqreturn

인터럽트는 cpu의 정상적인 스케줄링을 방해하고, 리소스를 점령합니다.<br>
따라서, <B>인터럽트는 매우 빠르고 간결하게 처리되어야 합니다.<B><br>
필요에 따라, 인터럽트 핸들러 안에 많은 동작을 넣어야 할 필요가 있을 수 있습니다.<br>
이 부분은 리눅스 커널에서 인터럽트 후처리 기법(IRQ스레드, worker 등)이 있으나, 여기서는 생략하겠습니다.<br>

인터럽트 핸들러는 복잡한 연산은 물론이고, 스케줄링 함수는 호출되어서는 안됩니다.<br>
핸들러는 모든 동작이 완료 되면 <span style="{{ site.code }}">IRQ_HANDLED</span> 을 호출합니다.<br>
(include/linux/irqreturn.h)
```
enum irqreturn {
    IRQ_NONE        = (0 << 0),
    IRQ_HANDLED     = (1 << 0),
    IRQ_WAKE_THREAD     = (1 << 1),
};
```
<br>

핸들러가 <span style="{{ site.code }}">IRQ_HANDLED</span> 를 return할 때마다, 커널은 <span style="{{ site.code }}">/proc/interrupts</span> 에 해당 인터럽트 카운트를 1씩 증가시킵니다.<br>

### callstack

ARM-A 아키텍처는 대부분의 인터럽트는 GIC(Generic Interrupt Controller)에서 관리합니다.<br>
따라서, 대부분의 callstack은 다음과 같습니다.
```
=> ...
=> __handle_irq_event_percpu
=> handle_irq_event_percpu
=> handle_irq_event
=>handle_fasteoi_irq
=>generic_handle_irq
=>__handle_domain_irq
=>gic_handle_irq
=>el1_irq
```
<br>

여기서 각 드라이버의 인터럽트 핸들러는 <span style="{{ site.code }}">handle_fasteoi_irq</span> 내에서 호출됩니다.<br>
드라이버 코드를 추적하다보면 다음과 같은 코드에서 인터럽트 핸들러가 호출됨을 알 수 있습니다.
```
irqreturn_t __handle_irq_event_percpu(struct irq_desc *desc, unsigned int *flags)
{
    irqreturn_t retval = IRQ_NONE;
    unsigned int irq = desc->irq_data.irq;
    struct irqaction *action;

    record_irq_time(desc);

    for_each_action_of_desc(desc, action) {
        irqreturn_t res;

        ...

        trace_irq_handler_entry(irq, action);
        res = action->handler(irq, action->dev_id);
        trace_irq_handler_exit(irq, action, res);

        ...

        switch (res) {
        case IRQ_WAKE_THREAD:

        ...

            __irq_wake_thread(desc, action);

            fallthrough;    /* to add to randomness */
        case IRQ_HANDLED:
            *flags |= action->flags;
            break;
        default:
            break;
        }

        retval |= res;
    }

    return retval;
}
```
<br>

인터럽트 핸들러를 호출하는 부분은 <span style="{{ site.code }}">res = action->handler(irq, action->dev_id);</span> 부분입니다.<br>
인터럽트 핸들러는 action으로 관리가 되고있고, action 구조체 멤버 handler를 호출하는 것을 확인할 수 있습니다.<br>

핸들러 return값에 따라 어떻게 진행되는지도 알 수 있습니다.<br>
<span style="{{ site.code }}">IRQ_WAKE_THREAD</span> return으로 인터럽트 후처리를 진행하고, 인터럽트 처리가 끝나면,<br>
fallthrough로 <span style="{{ site.code }}">IRQ_HANDLED</span> 를 검사하는 것도 확인할 수 있습니다.<br>
