---
permalink: /documents/aspice/smmu/smmu-architecture-design/
title: SMMU Software Architectural Design (SWE.2)
excerpt: Software architectural design for the SMMUv3 driver, targeting Linux Kernel 6.12.
toc: true
---

*Note: The images in this document were generated using AI.*

# 1 Module Overview

## 1.1 Purpose of Module
The <span style="{{ site.code }}">SMMU Software Architectural Design</span> defines the high-level structure and components of the SMMUv3 driver. It bridges the gap between software requirements (SWE.1) and detailed design (SWE.3), ensuring the system is robust, performant, and maintainable.

## 1.2 Role of Module
The SMMU module acts as the IOMMU driver for ARM64 systems. Its primary roles are:
*   **Translation Management:** Managing Page Tables (Stage 1/2) for DMA address translation.
*   **Isolation:** Enforcing access control by ensuring devices can only access mapped memory.
*   **Hardware Abstraction:** Encapsulating SMMUv3 hardware details (Queues, Tables) from the generic IOMMU subsystem.
*   **Fault Reporting:** Catching and reporting DMA faults (e.g., translation errors) to the kernel.

## 1.3 Boundary of Module
*   **Interfaces:** Interacts with the Linux IOMMU Subsystem (upper layer), ACPI/Device Tree (configuration), and SMMUv3 Hardware (lower layer).
*   **Scope:** Covers `arm-smmu-v3.c`, `io-pgtable-arm.c` (interaction), and associated headers.
*   **Exclusions:** Does not include the core Logic of the PCI subsystem or the generic IOMMU API itself.

# 2 Module Architecture

## 2.1 Implementation View
The implementation view shows the static code structure and file organization.

<img src="/assets/images/documents/aspice/smmu/smmu-implementation-view.png" alt="SMMU Implementation View" width="600">

### Role of Implementation elements

| Implementation Element | Role |
| :--- | :--- |
| **SMMUv3 Core (`arm-smmu-v3.c`)** | Main driver logic: Initialization, Queue Management, Command Issue. |
| **IO Page Table (`io-pgtable-arm.c`)** | Helper library for manipulating ARMv8/LPAE page table formats. |
| **SMMUv3 Header (`arm-smmu-v3.h`)** | Defines hardware register offsets, queue entry formats, and macros. |

## 2.2 Runtime View
The runtime view illustrates the dynamic behavior and interaction of objects during execution.

<img src="/assets/images/documents/aspice/smmu/smmu-runtime-view.png" alt="SMMU Runtime View" width="600">

### Role of runtime elements

| Runtime Element | Role |
| :--- | :--- |
| **SMMU Device (`arm_smmu_device`)** | Represents a physical SMMUv3 hardware instance. |
| **SMMU Domain (`arm_smmu_domain`)** | Represents an abstract IOMMU domain (container for mappings). |
| **Master Device (`arm_smmu_master`)** | Represents a client device (e.g., PCIe EP) attached to the SMMU. |
| **Command Queue** | Ring buffer for sending commands (TLBI, SYNC) to hardware. |

### Role of connector

| Connector | Role |
| :--- | :--- |
| **IOMMU Ops** | `iommu_ops` function table called by the generic IOMMU layer. |
| **MMIO** | Memory-Mapped I/O for register access. |
| **DMA** | Direct Memory Access for Queue consumption by hardware. |

### Runtime-to-Implementation Mapping

| Runtime View Elements | Implementation View Elements |
| :--- | :--- |
| **SMMU Device** | `struct arm_smmu_device` in `arm-smmu-v3.c` |
| **SMMU Domain** | `struct arm_smmu_domain` in `arm-smmu-v3.c` |
| **Page Table Ops** | `struct io_pgtable_ops` in `io-pgtable-arm.c` |

## 2.3 Allocation View
The allocation view maps software components to hardware units.
*   **SMMU Driver:** Runs on the CPU, managing system memory structures (Tables, Queues).
*   **Hardware Unit:** The SMMUv3 hardware fetches commands from memory and performs translations using cached TLBs and the in-memory Stream/Page Tables.

# 3 Module Interaction

## 3.1 Interaction: DMA Mapping & TLB Invalidation
This interaction describes the process when a driver maps a buffer for DMA.

**Map and Invalidate Sequence**
<img src="/assets/images/documents/aspice/smmu/smmu-interaction-map.png" alt="DMA Map Interaction" width="600">

**Flow:**
1.  Device Driver calls `dma_map_single()`.
2.  IOMMU Subsystem calls `arm_smmu_map()`.
3.  SMMU Driver updates Page Table (via `io-pgtable`).
4.  SMMU Driver issues `CMD_TLBI` to Command Queue.
5.  SMMU Driver issues `CMD_SYNC` and polls for completion.

# 4 Annex

## 4.1 Annex A. Alternative Architecture
### 4.1.1 Strict vs. Lazy Mode
*   **Strict Mode:** Issue TLB Invalidation (and Sync) immediately after every unmap. Secure but slower.
*   **Lazy Mode:** Defer TLB Invalidation until a threshold is reached or memory is reused. Faster but less secure (theoretical window for use-after-free).
*   **Selection:** The driver supports both, configurable via kernel command line. Default is Strict for safety/security.

## 4.2 Annex B. Alternative Architecture Evaluation
| ID | Quality Attribute | Score | Reason for Evaluation |
| :--- | :--- | :--- | :--- |
| **Alt-01** | Performance | High | Lazy mode significantly improves throughput for high-churn workloads. |
| **Alt-02** | Security | Low | Lazy mode leaves stale TLB entries window. |
| **Evaluation Result** | **Adopt (Configurable)** | Allow users to choose based on system requirements (Safety vs. Perf). |

## 4.3 Annex C. Selected Architectural Design
The selected design uses a **Queue-Based Command Interface** with support for **Strict Mode** by default.
*   **Justification:** Aligns with the ARM SMMUv3 specification and prioritizes correctness and isolation (Safety) over raw throughput, while retaining flexibility.

# 5 Terms and Abbreviations

| Terms and Abbreviations | Description |
| :--- | :--- |
| **TLBI** | TLB Invalidate |
| **IOVA** | Input Output Virtual Address |
| **LPAE** | Large Physical Address Extension |
| **STE** | Stream Table Entry |

# 6 References

| Documents Name | Version |
| :--- | :--- |
| SMMU Software Requirements Specification (SWE.1) | 1.0 |
| Linux Kernel Documentation (iommu) | 6.12 |
