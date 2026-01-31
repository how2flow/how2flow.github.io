---
permalink: /documents/aspice/smmu/smmu-functional-test-specification/
title: SMMU Software Functional Test Specification (SWE.4)
excerpt: Functional test specification for the SMMUv3 driver, verifying module functions.
toc: true
---

*Note: The images in this document were generated using AI.*

# 1 Overview
The <span style="{{ site.code }}">SMMU Software Functional Test Specification</span> defines the test cases to verify the functional correctness of the SMMUv3 driver modules. It aims to verify the functions defined in the Detailed Design (SWE.3) and ensure the module meets the functional requirements (SWE.1).

# 2 Test Environment

## 2.1 Basic Information
| Item | Description |
| :--- | :--- |
| **Test Level** | Module Functional Test (SWE.4) |
| **Test Framework** | KUnit (Kernel Unit Testing) |
| **Target Hardware** | QEMU (virt machine) |
| **Kernel Version** | Linux 6.12 |

## 2.2 System Configuration
<img src="/assets/images/documents/aspice/smmu/smmu-functional-test-env-diagram.png" alt="Test Environment Diagram" width="600">
The test environment utilizes the Linux Kernel's KUnit framework. The SMMU driver functions are tested in isolation (using mocks for hardware registers where necessary) running within a QEMU virtual machine.

# 3 Test Case

## 3.1 TC-SMMU-FUNC-01: Initialization Success
**Verify arm_smmu_device_probe Success Path**
*   **Traceability:** (SWE.3), REQ-SMMU-INIT-01 (SWE.1)
*   **Description:** Verify that `arm_smmu_device_probe` returns 0 (success) when valid ACPI/DT data is present and memory allocation succeeds.

| Input | Pre-Condition | Procedure | Expected Result |
| :--- | :--- | :--- | :--- |
| Valid `platform_device` struct | Kernel Booted, KUnit Loaded | Call `arm_smmu_device_probe(pdev)` | Function returns `0`. |

| Test Item | Input Data | Expected Result |
| :--- | :--- | :--- |
| **Return Value** | Valid `pdev` | `0` |
| **Device State** | N/A | `smmu->features` populated. |

## 3.2 TC-SMMU-FUNC-02: Map IOVA Success
**Verify arm_smmu_map Logic**
*   **Traceability:** (SWE.3), REQ-SMMU-MAP-01 (SWE.1)
*   **Description:** Verify that `arm_smmu_map` correctly writes a Page Table Entry (PTE) and issues a TLB invalidation.

| Input | Pre-Condition | Procedure | Expected Result |
| :--- | :--- | :--- | :--- |
| IOVA=0x1000, PA=0x2000, Size=4KB | Domain initialized | Call `arm_smmu_map(domain, 0x1000, 0x2000, 4096, PROT_READ)` | Function returns `0`. |

| Test Item | Input Data | Expected Result |
| :--- | :--- | :--- |
| **Return Value** | valid params | `0` |
| **Page Table** | IOVA 0x1000 | PTE contains PA 0x2000. |

## 3.3 TC-SMMU-FUNC-03: Map Invalid Size
**Verify arm_smmu_map Error Handling**
*   **Traceability:** (SWE.3)
*   **Description:** Verify that `arm_smmu_map` returns an error when an unsupported page size is requested.

| Input | Pre-Condition | Procedure | Expected Result |
| :--- | :--- | :--- | :--- |
| Size=3KB (Unaligned) | Domain initialized | Call `arm_smmu_map(domain, ..., 3072, ...)` | Function returns `-EINVAL`. |

| Test Item | Input Data | Expected Result |
| :--- | :--- | :--- |
| **Return Value** | Size=3072 | `-EINVAL` |

# 4 Terms and Abbreviations

| Terms and Abbreviations | Description |
| :--- | :--- |
| **KUnit** | Kernel Unit Testing Framework |
| **PTE** | Page Table Entry |

# 5 References

| Documents Name | Version |
| :--- | :--- |
| SMMU Software Requirements Specification (SWE.1) | 1.0 |
| SMMU Software Detailed Design (SWE.3) | 1.0 |
