---
permalink: /documents/aspice/gic/gic-functional-test-specification/
title: GIC Software Functional Test Specification (SWE.4)
excerpt: Functional test specification for the GICv3 and ITS driver, verifying module functions.
toc: true
---

*Note: The images in this document were generated using AI.*

# 1 Overview
The <span style="{{ site.code }}">GIC Software Functional Test Specification</span> defines the test cases to verify the functional correctness of the GICv3 and ITS driver modules. It aims to verify the functions defined in the Detailed Design (SWE.3) and ensure the module meets the functional requirements (SWE.1).

# 2 Test Environment

## 2.1 Basic Information
| Item | Description |
| :--- | :--- |
| **Test Level** | Module Functional Test (SWE.4) |
| **Test Framework** | KUnit (Kernel Unit Testing) |
| **Target Hardware** | QEMU (virt machine) |
| **Kernel Version** | Linux 6.12 |

## 2.2 System Configuration
<img src="/assets/images/documents/aspice/gic/gic-functional-test-env-diagram.png" alt="Test Environment Diagram" width="600">
The test environment utilizes the Linux Kernel's KUnit framework. The GIC driver functions are tested in isolation (or with mocks) running within a QEMU virtual machine managed by the host build system.

# 3 Test Case

## 3.1 TC-GIC-FUNC-01: GIC Initialization Success
**Verify gic_of_init Success Path**
*   **Traceability:** (SWE.3), REQ-GIC-INIT-01 (SWE.1)
*   **Description:** Verify that `gic_of_init` returns 0 (success) when a valid Device Tree node is provided.

| Input | Pre-Condition | Procedure | Expected Result |
| :--- | :--- | :--- | :--- |
| Valid `device_node` struct | Kernel Booted, KUnit Loaded | Call `gic_of_init(valid_node, NULL)` | Function returns `0`. |

| Test Item | Input Data | Expected Result |
| :--- | :--- | :--- |
| **Return Value** | `node` with `compatible="arm,gic-v3"` | `0` |
| **Global State** | N/A | `gic_data` initialized. |

## 3.2 TC-GIC-FUNC-02: GIC Initialization Failure
**Verify gic_of_init Failure Path**
*   **Traceability:** (SWE.3)
*   **Description:** Verify that `gic_of_init` returns an error code when an invalid Device Tree node is provided.

| Input | Pre-Condition | Procedure | Expected Result |
| :--- | :--- | :--- | :--- |
| Invalid `device_node` struct | Kernel Booted, KUnit Loaded | Call `gic_of_init(invalid_node, NULL)` | Function returns `-ENODEV` or similar. |

| Test Item | Input Data | Expected Result |
| :--- | :--- | :--- |
| **Return Value** | `node` = `NULL` | Negative Error Code |

## 3.3 TC-GIC-FUNC-03: ITS Device Allocation
**Verify its_alloc_device**
*   **Traceability:** (SWE.3), REQ-GIC-ITS-03 (SWE.1)
*   **Description:** Verify that `its_alloc_device` correctly allocates memory and assigns an ID.

| Input | Pre-Condition | Procedure | Expected Result |
| :--- | :--- | :--- | :--- |
| `dev_id` = 10, `nvecs` = 2 | ITS Initialized | Call `its_alloc_device(its_node, 10, 2)` | Function returns valid `struct its_device *`. |

| Test Item | Input Data | Expected Result |
| :--- | :--- | :--- |
| **Return Pointer** | `dev_id=10` | Non-NULL pointer |
| **Device ID** | `dev_id=10` | `device->device_id == 10` |

# 4 Terms and Abbreviations

| Terms and Abbreviations | Description |
| :--- | :--- |
| **KUnit** | Kernel Unit Testing Framework |
| **DUT** | Device Under Test |

# 5 References

| Documents Name | Version |
| :--- | :--- |
| GIC Software Requirements Specification (SWE.1) | 1.0 |
| GIC Software Detailed Design (SWE.3) | 1.0 |
