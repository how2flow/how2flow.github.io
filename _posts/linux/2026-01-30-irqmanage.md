---
categories:
- linux
excerpt: Interrupt management in the Linux kernel
header:
  teaser: /assets/images/posts/thumbnails/note.jpg
layout: single
tags:
- interrupt
title: '[Linux] IRQ management'
toc: true
redirect_from:
- /documents/linux/irqmanage/
- /legacy/irqmanage/
- /documents/linux/irqmanage
- /legacy/irqmanage
---
## Introduction

This is an explanation of how interrupt management works in the Linux kernel (ver = 5.10).<br>

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

`hwirq` is the actual hardware-specific interrupt number.<br>
The Linux kernel manages this by mapping it to a virtual interrupt number.<br>
Developers define hardware interrupt numbers in the <span style="{{ site.code }}">device-tree</span>.
```
	label: device@10000000 {
		status = "okay";
		compatible = "samsung,device";
		interrupts = <GIC_SPI 38 IRQ_TYPE_LEVEL_HIGH>;
		...
	}
```
<br>

Number 38 is the hardware interrupt number, and this information can be found in the SoC or device datasheet.<br>
In the Linux kernel, this is managed as <span style="{{ site.code }}">hwirq</span>.<br>


## IRQ Information Management

The Linux kernel manages IRQ information through various objects.<br>
Structures containing interrupt information include <span style="{{ site.code }}">irq_chip</span>, <span style="{{ site.code }}">irq_desc</span>, and <span style="{{ site.code }}">irq_data</span>.<br>

### irq_desc

<span style="{{ site.code }}">irq_desc</span> is the top-level descriptor for an IRQ line.<br>
It stores the overall state of the corresponding IRQ.<br>
It also maps the handler information connected to that IRQ.<br>

The basic structure looks like this (include/linux/irqdesc.h):
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

Because it contains <span style="{{ site.code }}">irq_data</span>, if you know the <span style="{{ site.code }}">irq_desc</span>, you can retrieve the <span style="{{ site.code }}">irq_data</span>.<br>
Conversely, you can use the <span style="{{ site.code }}">container_of</span> macro to get it.<br>
This is implemented in the kernel.<br>

### irq_data

Inside `irq_desc` (or within its hierarchy), this structure holds the actual hardware IRQ number and information needed to control the IRQ (like `irq_chip` and hardware-specific flags).<br>
It stores hardware-level IRQ numbers (hwirq), tree structure data (parent/child irq_data), and IRQ line options (masking, trigger types, etc.).<br>
It can also store SoC or architecture-dependent data required when invoking methods provided by the `irq_chip`.<br>

The basic structure looks like this (include/linux/irq.h):
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

Because it contains <span style="{{ site.code }}">irq_chip</span>, if you know the <span style="{{ site.code }}">irq_data</span>, you can retrieve the <span style="{{ site.code }}">irq_chip</span>.<br>
Conversely, you can use the <span style="{{ site.code }}">container_of</span> macro to get it.<br>
This is implemented in the kernel.<br>

### irq_chip

This is a table of function pointers that implements operations such as activating/deactivating hardware interrupts (mask/unmask), ack, and eoi (End Of Interrupt).<br>
It provides control functions for the corresponding IRQ line (or group).<br>
Examples include <span style="{{ site.code }}">irq_chip->irq_ack</span>, <span style="{{ site.code }}">irq_chip->irq_mask</span>, <span style="{{ site.code }}">irq_chip->irq_unmask</span>, and <span style="{{ site.code }}">irq_chip->irq_eoi</span>.<br>
Typically, different interrupt control logic is implemented for each hardware type, such as ARM GIC (Generic Interrupt Controller), GPIO-based interrupt controllers, or PCI interrupts.<br>

The basic structure looks like this (include/linux/irq.h):
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

In the Linux kernel, interrupt handler functions have the <span style="{{ site.code }}">irqreturn_t</span> type.<br>
The handler for each IRQ is managed within the <span style="{{ site.code }}">irqaction</span> of the <span style="{{ site.code }}">irq_desc</span>.<br>
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

Interrupts interrupt the CPU's normal scheduling and consume resources.<br>
Therefore, <B>interrupts must be processed very quickly and concisely.</B><br>
Sometimes, there may be a need to perform many operations inside an interrupt handler.<br>
The Linux kernel offers interrupt deferred processing mechanisms (such as threaded IRQs and worker threads) for this, but those will be omitted here.<br>

Interrupt handlers must not perform complex calculations or call scheduling functions.<br>
When the handler completes all its tasks, it returns <span style="{{ site.code }}">IRQ_HANDLED</span>.<br>
(include/linux/irqreturn.h)
```
enum irqreturn {
    IRQ_NONE        = (0 << 0),
    IRQ_HANDLED     = (1 << 0),
    IRQ_WAKE_THREAD     = (1 << 1),
};
```
<br>

Every time a handler returns <span style="{{ site.code }}">IRQ_HANDLED</span>, the kernel increments the corresponding interrupt count by 1 in <span style="{{ site.code }}">/proc/interrupts</span>.<br>

### callstack

In the ARM-A architecture, most interrupts are managed by the GIC (Generic Interrupt Controller).<br>
Therefore, a typical callstack looks like this:
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

Here, each driver's interrupt handler is invoked within <span style="{{ site.code }}">handle_fasteoi_irq</span>.<br>
By tracing the driver code, you can see that the interrupt handler is called in code similar to the following:
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

The part that actually calls the interrupt handler is <span style="{{ site.code }}">res = action->handler(irq, action->dev_id);</span>.<br>
You can see that interrupt handlers are managed via the `action` structure, and it invokes the `handler` member of the `action` structure.<br>

It also shows how the execution proceeds based on the handler's return value.<br>
It handles deferred interrupt processing if <span style="{{ site.code }}">IRQ_WAKE_THREAD</span> is returned, and then falls through to check for <span style="{{ site.code }}">IRQ_HANDLED</span> when interrupt processing is complete.<br>
