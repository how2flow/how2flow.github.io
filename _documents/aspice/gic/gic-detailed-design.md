---
permalink: /documents/aspice/gic/gic-detailed-design/
title: GIC Software Detailed Design (SWE.3)
excerpt: Software detailed design for the GICv3 and ITS driver, targeting Linux Kernel 6.12.
toc: true
---

*Note: The images in this document were generated using AI.*

# 1 Overview
The <span style="{{ site.code }}">GIC Software Detailed Design</span> specifies the internal logic, data structures, and algorithms of the GICv3 and ITS driver components. It provides a blueprint for implementation (SWE.4) and ensures that each software unit is verifiable and traceable to the architectural design (SWE.2).

# 2 Component: GICv3 Core

## 2.1 Type Definition
| Type Name | Description |
| :--- | :--- |
| `struct gic_chip_data` | Main data structure holding GIC instance state (Distributor base, Redistributor regions). |
| `struct gic_kvm_info` | Structure to pass GIC information to KVM hypervisor. |

## 2.2 Macro Definition
| Macro Name | Description |
| :--- | :--- |
| `GICD_CTLR` | Offset for Distributor Control Register. |
| `GICR_WAKER` | Offset for Redistributor Waker Register. |

## 2.3 Global Variables
| Declaration | Type | Description |
| :--- | :--- | :--- |
| `gic_data` | `struct gic_chip_data` | Global instance of the GICv3 driver data. |

## 2.4 Function Specification

### 2.4.1 `gic_of_init`
**GICv3 Initialization via Device Tree**
*   **Property:** Synchronous, Non-reentrant
*   **Description:** Entry point for GICv3 initialization when booting with Device Tree. Probes hardware, maps registers, and registers the IRQ domain.

<img src="/assets/images/documents/aspice/gic/gic-func-gic-of-init.png" alt="Call Graph: gic_of_init" width="600">

| Item | Description |
| :--- | :--- |
| **Function Scope** | Global (init) |
| **Parameters** | `struct device_node *node`, `struct device_node *parent` |
| **Return** | `0` on success, negative error code on failure. |

# 3 Component: ITS Driver

## 3.1 Type Definition
| Type Name | Description |
| :--- | :--- |
| `struct its_node` | Represents a physical ITS hardware block. |
| `struct its_device` | Represents a PCI/Platform device connected to an ITS. |

## 3.2 Global Variables
| Declaration | Type | Description |
| :--- | :--- | :--- |
| `its_list` | `struct list_head` | List of all probed ITS nodes in the system. |

## 3.3 Function Specification

### 3.3.1 `its_init`
**ITS Subsystem Initialization**
*   **Property:** Synchronous
*   **Description:** Probes all ITS nodes defined in firmware, allocates command queues, and enables the ITS.

<img src="/assets/images/documents/aspice/gic/gic-func-its-init.png" alt="Call Graph: its_init" width="600">

| Item | Description |
| :--- | :--- |
| **Function Scope** | Global (init) |
| **Parameters** | None |
| **Return** | `0` on success. |

### 3.3.2 `its_alloc_device`
**Allocate ITS Device Resources**
*   **Property:** Synchronous, Reentrant
*   **Description:** Allocates a unique Device ID and sets up the Device Table entry in the ITS.

| Item | Description |
| :--- | :--- |
| **Function Scope** | Internal |
| **Parameters** | `struct its_node *its`, `u32 dev_id`, `int nvecs` |
| **Return** | Pointer to `struct its_device` or NULL. |

# 4 Annex

## 4.1 Annex A. Evaluation Criteria for Detailed Software Design
The design is evaluated based on HIS metrics:
1.  **Complexity:** Cyclomatic Complexity <= 10.
2.  **Coupling:** Calling Functions <= 5, Called Functions <= 7.

# 5 Terms and Abbreviations

| Terms and Abbreviations | Description |
| :--- | :--- |
| **DT** | Device Tree |
| **KVM** | Kernel-based Virtual Machine |

# 6 References

| Documents Name | Version |
| :--- | :--- |
| GIC Software Architectural Design (SWE.2) | 1.0 |
| Linux Kernel Source (drivers/irqchip/irq-gic-v3.c) | 6.12 |
