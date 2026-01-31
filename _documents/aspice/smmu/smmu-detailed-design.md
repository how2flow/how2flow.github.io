---
permalink: /documents/aspice/smmu/smmu-detailed-design/
title: SMMU Software Detailed Design (SWE.3)
excerpt: Software detailed design for the SMMUv3 driver, targeting Linux Kernel 6.12.
toc: true
---

*Note: The images in this document were generated using AI.*

# 1 Overview
The <span style="{{ site.code }}">SMMU Software Detailed Design</span> specifies the internal logic, data structures, and algorithms of the SMMUv3 driver components. It provides a blueprint for implementation (SWE.4) and ensures that each software unit is verifiable and traceable to the architectural design (SWE.2).

# 2 Component: SMMUv3 Core

## 2.1 Type Definition
| Type Name | Description |
| :--- | :--- |
| `struct arm_smmu_device` | Main data structure holding SMMU hardware state (features, queues, strtab). |
| `struct arm_smmu_cmdq_ent` | Structure representing a single command queue entry (opcode + operands). |

## 2.2 Macro Definition
| Macro Name | Description |
| :--- | :--- |
| `CMDQ_ERR_CERROR_NONE_IDX` | Index for 'no error' in command queue error reporting. |
| `STRTAB_STE_0_CFG_BYPASS` | Configuration value to set an STE to bypass translation. |

## 2.3 Global Variables
| Declaration | Type | Description |
| :--- | :--- | :--- |
| `arm_smmu_ops` | `struct iommu_ops` | Function pointers for the generic IOMMU subsystem interface. |

## 2.4 Function Specification

### 2.4.1 `arm_smmu_device_probe`
**SMMUv3 Device Initialization**
*   **Property:** Synchronous, Non-reentrant
*   **Description:** Entry point for SMMUv3 initialization. Detects hardware features, allocates queues/tables, and registers the device.

<img src="/assets/images/documents/aspice/smmu/smmu-func-probe.png" alt="Call Graph: arm_smmu_device_probe" width="600">

| Item | Description |
| :--- | :--- |
| **Function Scope** | Global (init) |
| **Parameters** | `struct platform_device *pdev` |
| **Return** | `0` on success, negative error code on failure. |

### 2.4.2 `arm_smmu_map`
**Map IOVA to Physical Address**
*   **Property:** Synchronous, Reentrant
*   **Description:** Maps a virtual address range to a physical address range in the page tables.

<img src="/assets/images/documents/aspice/smmu/smmu-func-map.png" alt="Call Graph: arm_smmu_map" width="600">

| Item | Description |
| :--- | :--- |
| **Function Scope** | Internal (via ops) |
| **Parameters** | `struct iommu_domain *domain`, `unsigned long iova`, `phys_addr_t paddr`, `size_t size`, `int prot` |
| **Return** | `0` on success, or error code. |

# 3 Annex

## 3.1 Annex A. Evaluation Criteria for Detailed Software Design
The design is evaluated based on HIS metrics:
1.  **Complexity:** Cyclomatic Complexity <= 10.
2.  **Coupling:** Calling Functions <= 5, Called Functions <= 7.

# 4 Terms and Abbreviations

| Terms and Abbreviations | Description |
| :--- | :--- |
| **IOVA** | Input Output Virtual Address |
| **STE** | Stream Table Entry |

# 5 References

| Documents Name | Version |
| :--- | :--- |
| SMMU Software Architectural Design (SWE.2) | 1.0 |
| Linux Kernel Source (drivers/iommu/arm/arm-smmu-v3/arm-smmu-v3.c) | 6.12 |
