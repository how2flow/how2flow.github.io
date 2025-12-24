---
permalink: /documents/gic-v3-v4/
title: GICv3 & GICv4
excerpt: "Generic Interrupt Controller v3 and v4 Architecture"
comments: false
toc: true
---

## GIC Overview

GIC stands for 'Generic Interrupt Controller'. For a summary, please refer to [GIC summary](/documents/gic-summary).<br>
The GIC is an ARM-specific interrupt controller provided by ARM.<br>
This document covers <B>GICv3</B> and its virtualization extension, <B>GICv4</B>.<br>

A representative software example is the Linux Kernel.<br>
Please refer to [IRQ management in Linux kernel](/posts/linux_irq_manage) for related content.<br>

### Introduction

An Interrupt Controller is literally a device that controls interrupts.<br>
When it receives an interrupt signal from an external device,<br>
it plays the role of delivering it to the appropriate PE (Core).<br>
<br>

**GICv3** introduced significant scalability improvements (LPIs, affinity routing) and a System Register interface for the CPU.<br>
**GICv4** builds upon GICv3, adding hardware support for **Direct Injection** of virtual interrupts (vLPIs) to Virtual Processing Elements (vPEs), significantly reducing virtualization overhead.<br>
<br>

The components of the GIC are as follows:
```
1. Distributor (GICD)
2. Redistributor (GICR)
3. CPU-INTERFACE (ICC)
4. ITS (optional, GICv3+)
```
<br>

## GIC Components

Broadly, there are:<br>
<span style="{{ site.code }}">Distributor</span> , (<B>SPIs</B> route, global configuration)<br>
<span style="{{ site.code }}">Redistributor</span> , (<B>SGIs</B>, <B>PPIs</B>, <B>LPIs</B> route, per-core management)<br>
<span style="{{ site.code }}">CPU-INTERFACE</span> , (Interface to the PE)<br>
<span style="{{ site.code }}">ITS</span> , (Translate <B>LPIs</B>).<br>

### Distributor (GICD)

The Distributor performs the role of distributing SPIs requested by on-chip devices or external devices.<br>
It manages interrupt priorities and routing for SPIs.<br>
It is **Memory-Mapped**.<br>

### Redistributor (GICR)

The Redistributor plays the role of handling SGIs, PPIs, and LPIs.<br>
It is located closer to the Core (PE) and allows for distributed design.<br>
For LPIs, it accesses memory where the pending/configuration tables are stored.<br>
It contains the <span style="{{ site.code }}">GICR_WAKER</span> register for power management.<br>
It is **Memory-Mapped**.<br>

### CPU-INTERFACE (ICC)

The CPU interface delivers the physical interrupt signal (IRQ/FIQ) to the Core and handles the Core's acknowledgment (ACK) and End-of-Interrupt (EOI) signals.<br>

In GICv2, it was memory-mapped.<br>
<U>From GICv3, the CPU Interface is accessed via **System Registers**.</U><br>
This reduces bus traffic and enables faster context switching.<br>
To access these registers in the Trace32 debugger, you must use the <span style="{{ site.code }}">SPR:</span> class.<br>
Examples: `ICC_IAR1_EL1` (Interrupt Acknowledge), `ICC_PMR_EL1` (Priority Mask), `ICC_EOIR1_EL1` (End of Interrupt).<br>

### ITS (Interrupt Translation Service)

It plays the role of translating the messages of message-based interrupts (MSIs).<br>
It is supported from GICv3 (optional but common in PCIe systems).<br>
It translates `(DeviceID, EventID)` into `(Collection, INTID)` using tables in memory.<br>

DT: Device Table (DeviceID -> ITT Base)<br>
ITT: Interrupt Translation Table (EventID -> INTID, CollectionID)<br>
CT: Collection Table (CollectionID -> Target Redistributor)<br>

<B>The translated information is loaded into system memory.</B>
```
This information must be verified through actual kernel code analysis.
```
<br>

## Interrupts

The status of an interrupt is determined by signals from the GIC and the Core.<br>
'Inactive', 'Pending', 'Active', 'Active & Pending'<br>
The interrupt status change is determined by the interrupt type:
```
- Level sensitive
- Edge trigger
```
<br>

Interrupt types are optional, but SGI and LPI are mostly Edge triggered.<br>
Regarding Level sensitive, SPI is <span style="{{ site.code }}">Active HIGH</span>, and PPI is <span style="{{ site.code }}">Active LOW</span>.<br>

<div style="text-align: center;">
  <img src="/assets/images/documents/gic/gic_level_sens.png" alt="gic_level_sens" width="640" height="480"><br>
  <span style="{{ site.img }}">Picture [1] GIC level sensitive</span>
</div>
<br>

<div style="text-align: center;">
  <img src="/assets/images/documents/gic/gic_edge.png" alt="gic_edge" width="640" height="480"><br>
  <span style="{{ site.img }}">Picture [2] GIC edge trigger</span>
</div>
<br>

### Interrupt Groups and Security

Interrupts are managed in groups for security (TrustZone):
- **Group 0**: Secure interrupts (FIQ to EL3).
- **Secure Group 1**: Secure interrupts (IRQ/FIQ to Secure EL1).
- **Non-secure Group 1**: Non-secure interrupts (IRQ to Non-secure EL1/EL2). Typical Linux interrupts.

### Interrupt Types

#### SGIs (Software Generated Interrupts)
- IDs: 0-15
- Used for Inter-Processor Communication (IPI).
- Classified as hardware interrupts in Linux.

#### PPIs (Private Peripheral Interrupts)
- IDs: 16-31
- Private to a specific core (e.g., Generic Timer, PMU).
- **ID 25** is typically used for the **Virtual Maintenance Interrupt (VMI)** in GICv3/v4 virtualization.

#### SPIs (Shared Peripheral Interrupts)
- IDs: 32 - 1019
- Shared by all cores. Routing is configured by the Distributor.
- Verify interrupts using `/proc/interrupts` and Trace32 (checking `ISACTIVR` and `ISPENDR`).

#### LPIs (Locality-specific Peripheral Interrupts)
- IDs: 8192 and above.
- Message-based (MSI/MSI-X).
- **Always Non-secure Group 1**.
- **Edge-Triggered** only.
- Configuration and Pending state are stored in **Memory Tables**, not registers.

**LPI Check conditions:**
```
/* LPI configs */
1. GICD_CTLR.DS == ? (0 or 1)
2. GICR_CTLR.EnableLPIs == 1
3. GICR_TYPER.CommonLPIAff (Optional)

/* Address infomation */
1. GITS_BASER (memory structure)
2. GITS_CBASER (ITS command queue)

/* LPI tables */
1. GICR_PROPBASER (Configuration Table)
2. GICR_PENDBASER (Pending Table)
```

## GICv4: Direct Injection

**GICv4** is an extension designed to optimize virtualization.<br>

In **GICv3**, injecting a virtual interrupt (vIRQ) to a Virtual Machine (VM) requires:
1. Physical interrupt traps to Hypervisor (EL2).
2. Hypervisor writes to List Registers (LRs) in the GIC.
3. GIC asserts vIRQ to the vCPU.
This "Trap-and-Emulate" model has high overhead.<br>

**GICv4** allows **Direct Injection**:
1. ITS translates an incoming MSI directly into a **vLPI** (Virtual LPI).
2. The hardware checks if the target **vPE** (Virtual PE) is scheduled.
3. If scheduled, the hardware injects the vLPI directly to the vCPU without Hypervisor intervention.
4. If not scheduled, it is recorded in a **vPE Table** in memory.

<div style="text-align: center;">
  <img src="/assets/images/documents/gic/gicv4-direct-injection.png" alt="GICv4 Direct Injection" width="640" height="480"><br>
  <span style="{{ site.img }}">Picture [3] GICv4 Direct Injection</span>
</div>
<br>

This significantly reduces CPU overhead for network and storage virtualization.

## Power Management

GICv3/v4 introduces a handshake mechanism for power management using the **GICR_WAKER** register in the Redistributor.<br>

1. **ProcessorSleep**: When a core is about to enter a low-power state (e.g., idle), it sets the `ProcessorSleep` bit.
2. The GIC ensures all pending interrupts for that core are forwarded or handled.
3. **ChildrenAsleep**: When the GIC is ready, it asserts `ChildrenAsleep`.
4. The core can now safely power down.

On wake-up, the core clears `ProcessorSleep`, and the GIC resumes interrupt delivery.

## Programmer's Model Summary

| Component | Access Type | Note |
|---|---|---|
| **Distributor** | Memory Mapped | Global settings, SPI routing. |
| **Redistributor** | Memory Mapped | Per-core settings, SGI/PPI/LPI management. |
| **ITS** | Memory Mapped | Translation services (MSI). |
| **CPU Interface** | **System Registers** | Accessed via `MSR`/`MRS`. Low latency. |

**Important System Registers:**
- `ICC_PMR_EL1`: Priority Mask Register (Set interrupt threshold).
- `ICC_IAR1_EL1`: Interrupt Acknowledge Register (Read to get INTID).
- `ICC_EOIR1_EL1`: End of Interrupt Register (Write INTID to finish).
- `ICC_CTLR_EL1`: Control Register.

---
**References:**
- [ARM Generic Interrupt Controller Architecture Specification GICv3 and GICv4 (IHI 0069)](https://developer.arm.com/documentation/ihi0069/latest/)
- [GICv3 and GICv4 Software Overview](https://developer.arm.com/documentation/dai0492/latest/)
