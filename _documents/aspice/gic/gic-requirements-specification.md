---
permalink: /documents/aspice/gic/gic-requirements/
title: GIC Software Requirements Specification (SWE.1)
excerpt: Software requirements for the GICv3 and ITS driver, targeting Linux Kernel 6.12.
toc: true
---

*Note: The images in this document were generated using AI.*

# 1 Overview

## 1.1 Purpose and Role of Module

### 1.1.1 Module Overview
The <span style="{{ site.code }}">Generic Interrupt Controller (GIC)</span> driver is a critical component in ARM-based SoCs, managing interrupts from peripherals to CPU cores. This document specifies the software requirements for the GICv3 and ITS (Interrupt Translation Service) driver, targeting <span style="{{ site.code }}">Linux Kernel 6.12</span>. The module ensures that hardware interrupts are correctly routed, prioritized, and handled by the operating system.

### 1.1.2 Internal and External Factors of Module

| Category | Factors |
| :--- | :--- |
| **Internal Factors** | • **Interrupt Handling Logic:** Mechanisms to acknowledge, mask, and EOI interrupts.<br>• **Register State Machine:** Managing GICD/GICR/ITS register states.<br>• **Data Structures:** `gic_chip_data`, `its_node`, bitmap allocators. |
| **External Factors** | • **CPU Cores:** Interface with CPU for exception handling.<br>• **Peripherals:** Devices generating SPIs, PPIs, and MSIs.<br>• **Power Management Subsystem:** System suspend/resume signals.<br>• **Firmware:** ACPI tables or Device Tree providing topology info. |

## 1.2 Hardware Diagram

<img src="/assets/images/documents/aspice/gic/gic-hardware-diagram.png" alt="GIC Hardware Diagram Placeholder" width="600">

## 1.3 Scope of Module
This Software Requirements Specification covers the initialization, configuration, and runtime management of the GICv3 and ITS hardware blocks. It includes:
*   Detection of GIC hardware via ACPI/DT.
*   Resource allocation for LPIs and MSIs.
*   Power management state saving/restoring.
*   SMP affinity management.

## 1.4 Software Context Diagram

<img src="/assets/images/documents/aspice/gic/gic-sw-context-diagram.png" alt="GIC Software Context Diagram Placeholder" width="600">

# 2 Feature & Specification

## 2.1 Feature list

| No | Feature | Details | Remarks |
| :--- | :--- | :--- | :--- |
| 1 | **Initialization** | Probing GICv3/ITS hardware and initializing base addresses. | Boot time |
| 2 | **Interrupt Routing** | Routing SPIs, PPIs, and LPIs to target CPUs. | Runtime |
| 3 | **ITS Support** | Managing MSI-to-LPI translation for PCI/Platform devices. | Runtime |
| 4 | **Power Management** | Suspend and Resume support for GIC context. | System Sleep |

## 2.2 Specification

| No | Item | Specification | Remarks |
| :--- | :--- | :--- | :--- |
| 1 | **Architecture** | ARM Generic Interrupt Controller Architecture Specification GICv3/v4 | ARM IHI 0069 |
| 2 | **Kernel Version** | Linux Kernel 6.12 LTS | |
| 3 | **Interface** | Linux `irq_chip` and `irq_domain` APIs | |

# 3 Requirements

## 3.1 Functional Requirements

### 3.1.1 Initialization (REQ-GIC-INIT)

| Attribute | Description |
| :--- | :--- |
| **Feature** | Initialization |
| **Quality Attribute** | Reliability |
| **Input/Stimulus** | System Boot, ACPI MADT / Device Tree |
| **Output/Response** | GICD/GICR mapped, Interrupts enabled, success log in dmesg. |
| **Pass/Fail Condition** | Driver loads without error; `/proc/interrupts` shows GIC controller. |
| **Verification Method** | Unit Test / Integration Test |
| **Verification Environment** | QEMU / EVB Target |

**Detailed Requirements:**
*   **REQ-GIC-INIT-01:** The driver SHALL detect the GIC version from the <span style="{{ site.code }}">ACPI</span> MADT table or <span style="{{ site.code }}">Device Tree</span> compatible string `arm,gic-v3`.
*   **REQ-GIC-INIT-02:** The driver SHALL map the Distributor (<span style="{{ site.code }}">GICD</span>) base address.
*   **REQ-GIC-INIT-03:** The driver SHALL discover and map Redistributors (<span style="{{ site.code }}">GICR</span>) for all online CPUs.

### 3.1.2 Interrupt Routing (REQ-GIC-ROUTE)

| Attribute | Description |
| :--- | :--- |
| **Feature** | Interrupt Routing |
| **Quality Attribute** | Functionality |
| **Input/Stimulus** | `request_irq()`, `irq_set_affinity()` calls |
| **Output/Response** | Interrupt delivered to correct CPU core. |
| **Pass/Fail Condition** | Interrupt counts increment on expected CPU in `/proc/interrupts`. |
| **Verification Method** | System Test |
| **Verification Environment** | EVB Target |

**Detailed Requirements:**
*   **REQ-GIC-ROUTE-01:** The driver SHALL support routing of Shared Peripheral Interrupts (<span style="{{ site.code }}">SPIs</span>) to any online CPU.
*   **REQ-GIC-ROUTE-02:** The driver SHALL support setting the affinity of an interrupt via `irq_set_affinity`.
*   **REQ-GIC-ROUTE-03:** The driver SHALL route Private Peripheral Interrupts (<span style="{{ site.code }}">PPIs</span>) to the specific requesting CPU.

### 3.1.3 ITS Support (REQ-GIC-ITS)

| Attribute | Description |
| :--- | :--- |
| **Feature** | ITS Support |
| **Quality Attribute** | Functionality |
| **Input/Stimulus** | PCI Device Enumeration (MSI Request) |
| **Output/Response** | LPI allocated, Translation Table updated. |
| **Pass/Fail Condition** | MSI interrupts function correctly for PCIe devices. |
| **Verification Method** | Integration Test |
| **Verification Environment** | EVB Target |

**Detailed Requirements:**
*   **REQ-GIC-ITS-01:** The driver SHALL detect <span style="{{ site.code }}">ITS</span> nodes described in firmware.
*   **REQ-GIC-ITS-02:** The driver SHALL allocate command queues for each <span style="{{ site.code }}">ITS</span>.
*   **REQ-GIC-ITS-03:** The driver SHALL manage the translation table to map DeviceID+EventID to <span style="{{ site.code }}">LPIs</span>.

### 3.1.4 Power Management (REQ-GIC-PM)

| Attribute | Description |
| :--- | :--- |
| **Feature** | Power Management |
| **Quality Attribute** | Reliability |
| **Input/Stimulus** | `pm_suspend` command |
| **Output/Response** | GIC state saved to RAM; System enters sleep. |
| **Pass/Fail Condition** | System resumes and interrupts function immediately. |
| **Verification Method** | System Test |
| **Verification Environment** | EVB Target |

**Detailed Requirements:**
*   **REQ-GIC-PM-01:** The driver SHALL save the state of <span style="{{ site.code }}">GICD</span> and <span style="{{ site.code }}">GICR</span> registers upon system suspend.
*   **REQ-GIC-PM-02:** The driver SHALL restore the state of <span style="{{ site.code }}">GICD</span> and <span style="{{ site.code }}">GICR</span> registers upon system resume.

## 3.2 Non-Functional Requirements

### 3.2.1 Quality Requirements

| Attribute | Description |
| :--- | :--- |
| **Feature** | N/A |
| **Quality Attribute** | Performance (Latency) |
| **Input/Stimulus** | High-frequency interrupt generation |
| **Output/Response** | Interrupt service routine execution. |
| **Pass/Fail Condition** | Latency does not exceed 10 microseconds for LPIs. |
| **Verification Method** | Performance Measurement |
| **Verification Environment** | Real-time Kernel Setup |

**Detailed Requirements:**
*   **REQ-GIC-NF-01:** The driver initialization MUST complete before secondary CPUs are brought up.
*   **REQ-GIC-NF-02:** The interrupt latency introduced by the driver MUST NOT exceed 10 microseconds for <span style="{{ site.code }}">LPIs</span>.

### 3.2.2 Constraints

| Attribute | Description |
| :--- | :--- |
| **Feature** | N/A |
| **Constraint** | Interface Compliance |
| **Description** | The driver MUST register with the Linux IRQ subsystem using standard APIs (`irq_domain`). |

**Detailed Requirements:**
*   **REQ-GIC-IF-01:** The driver MUST register with the <span style="{{ site.code }}">Linux</span> IRQ subsystem using `irq_domain_add_*`.

# 4 Annex

## 4.1 Annex A. Domain Model
<img src="/assets/images/documents/aspice/gic/gic-domain-model.png" alt="GIC Domain Model" width="600">

## 4.2 Annex B. Quality Scenario
**Scenario: ITS Interrupt Translation & Latency**
<img src="/assets/images/documents/aspice/gic/gic-quality-scenario.png" alt="ITS Interrupt Handling Scenario" width="600">

## 4.3 Annex C. Quality Scenario Analysis
*Analysis of the above scenarios.*

# 5 Terms and Abbreviations

| Term | Description |
| :--- | :--- |
| **GIC** | Generic Interrupt Controller |
| **ITS** | Interrupt Translation Service |
| **SPI** | Shared Peripheral Interrupt |
| **PPI** | Private Peripheral Interrupt |
| **LPI** | Locality-specific Peripheral Interrupt |
| **MSI** | Message Signaled Interrupt |
| **ACPI** | Advanced Configuration and Power Interface |
| **DT** | Device Tree |

# 6 References

| Document Name | Version |
| :--- | :--- |
| ARM GICv3 Architecture Specification | IHI 0069H |
| Linux Kernel Source Code | 6.12 |
