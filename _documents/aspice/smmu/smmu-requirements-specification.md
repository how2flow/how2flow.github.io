---
permalink: /documents/aspice/smmu/smmu-requirements-specification/
title: SMMU Software Requirements Specification (SWE.1)
excerpt: Software requirements for the SMMUv3 driver, targeting Linux Kernel 6.12.
toc: true
---

*Note: The images in this document were generated using AI.*

# 1 Overview

## 1.1 Purpose and Role of Module

### 1.1.1 Module Overview
The <span style="{{ site.code }}">System Memory Management Unit (SMMU)</span> driver is responsible for managing the SMMUv3 hardware, which provides address translation and access control for DMA-capable master devices. This document specifies the software requirements for the SMMUv3 driver, targeting <span style="{{ site.code }}">Linux Kernel 6.12</span>. It ensures isolation between devices and supports virtualization (Stage 1/Stage 2 translation).

### 1.1.2 Internal and External Factors of Module

| Category | Factors |
| :--- | :--- |
| **Internal Factors** | • **Translation Tables:** Management of page tables (LPAE/ARMv8 format).<br>• **Queue Management:** Circular buffer logic for Command and Event queues.<br>• **Stream Table:** Configuration of Stream Table Entries (STEs). |
| **External Factors** | • **Master Devices:** PCIe and Platform devices initiating DMA.<br>• **IOMMU Subsystem:** Linux generic IOMMU layer providing API calls.<br>• **Firmware:** ACPI (IORT) or Device Tree description of SMMU topology. |

## 1.2 Hardware Diagram

<img src="/assets/images/documents/aspice/smmu/smmu-hardware-diagram.png" alt="SMMU Hardware Diagram" width="600">

## 1.3 Scope of Module
This Software Requirements Specification covers the initialization, configuration, and runtime operation of the SMMUv3 driver. It includes:
*   Detection and initialization of SMMUv3 hardware.
*   Management of IOMMU domains (allocation, attach/detach).
*   Map and Unmap operations for DMA addresses.
*   Handling of Context Faults and Events.

## 1.4 Software Context Diagram

<img src="/assets/images/documents/aspice/smmu/smmu-sw-context-diagram.png" alt="SMMU Software Context Diagram" width="600">

# 2 Feature & Specification

## 2.1 Feature list

| No | Feature | Details | Remarks |
| :--- | :--- | :--- | :--- |
| 1 | **Initialization** | Probing SMMUv3, allocating queues (CMD, EVT, PRI). | Boot time |
| 2 | **Domain Management** | Creating/Destroying IOMMU domains, attaching devices. | Runtime |
| 3 | **Translation** | Mapping IOVA to Physical Addresses (Page Tables). | Runtime |
| 4 | **Fault Handling** | Reporting and recovering from translation faults. | Runtime |

## 2.2 Specification

| No | Item | Specification | Remarks |
| :--- | :--- | :--- | :--- |
| 1 | **Architecture** | ARM System Memory Management Unit Architecture Specification v3 | ARM IHI 0070 |
| 2 | **Kernel Version** | Linux Kernel 6.12 LTS | |
| 3 | **Interface** | Linux `iommu_ops` API | |

# 3 Requirements

## 3.1 Functional Requirements

### 3.1.1 Initialization (REQ-SMMU-INIT)

| Attribute | Description |
| :--- | :--- |
| **Feature** | Initialization |
| **Quality Attribute** | Reliability |
| **Input/Stimulus** | System Boot, ACPI IORT / Device Tree |
| **Output/Response** | Hardware reset, Queues enabled, Driver registered. |
| **Pass/Fail Condition** | Driver probes successfully; `dmesg` shows SMMU initialized. |
| **Verification Method** | Unit Test / Integration Test |
| **Verification Environment** | QEMU / EVB Target |

**Detailed Requirements:**
*   **REQ-SMMU-INIT-01:** The driver SHALL detect SMMUv3 hardware via <span style="{{ site.code }}">ACPI IORT</span> or <span style="{{ site.code }}">Device Tree</span>.
*   **REQ-SMMU-INIT-02:** The driver SHALL allocate and initialize the <span style="{{ site.code }}">Command Queue</span> and <span style="{{ site.code }}">Event Queue</span> in system memory.
*   **REQ-SMMU-INIT-03:** The driver SHALL allocate the <span style="{{ site.code }}">Stream Table</span> (Linear or 2-level) based on hardware capability.

### 3.1.2 Translation & Mapping (REQ-SMMU-MAP)

| Attribute | Description |
| :--- | :--- |
| **Feature** | Translation |
| **Quality Attribute** | Functionality |
| **Input/Stimulus** | `iommu_map()` call |
| **Output/Response** | Page Table Entry (PTE) written, TLB invalidated. |
| **Pass/Fail Condition** | Device can access physical memory via IOVA. |
| **Verification Method** | System Test |
| **Verification Environment** | EVB Target |

**Detailed Requirements:**
*   **REQ-SMMU-MAP-01:** The driver SHALL support mapping of 4KB, 64KB, and 2MB pages (depending on granule).
*   **REQ-SMMU-MAP-02:** The driver SHALL issue a <span style="{{ site.code }}">CMD_TLBI</span> (TLB Invalidate) command to the SMMU after modifying page tables.
*   **REQ-SMMU-MAP-03:** The driver SHALL support <span style="{{ site.code }}">Stage 1</span> (VA->PA) translation for standard DMA.

### 3.1.3 Device Attachment (REQ-SMMU-ATTACH)

| Attribute | Description |
| :--- | :--- |
| **Feature** | Domain Management |
| **Quality Attribute** | Functionality |
| **Input/Stimulus** | `iommu_attach_device()` call |
| **Output/Response** | Stream Table Entry (STE) updated to point to Context Descriptor (CD). |
| **Pass/Fail Condition** | Device traffic is routed through the assigned domain. |
| **Verification Method** | Integration Test |
| **Verification Environment** | EVB Target |

**Detailed Requirements:**
*   **REQ-SMMU-ATTACH-01:** The driver SHALL write a valid <span style="{{ site.code }}">STE</span> pointing to the domain's Context Descriptor when a device is attached.
*   **REQ-SMMU-ATTACH-02:** The driver SHALL bypass translation (STE.Config=Bypass) if the device is attached to an Identity domain.

### 3.1.4 Fault Handling (REQ-SMMU-FAULT)

| Attribute | Description |
| :--- | :--- |
| **Feature** | Fault Handling |
| **Quality Attribute** | Reliability |
| **Input/Stimulus** | Context Fault (Hardware Interrupt) |
| **Output/Response** | Event Queue read, Fault reported to kernel. |
| **Pass/Fail Condition** | Fault details (StreamID, IOVA) logged in kernel ring buffer. |
| **Verification Method** | System Test |
| **Verification Environment** | EVB Target |

**Detailed Requirements:**
*   **REQ-SMMU-FAULT-01:** The driver SHALL handle the <span style="{{ site.code }}">Event Queue</span> interrupt.
*   **REQ-SMMU-FAULT-02:** The driver SHALL decode the event packet and report the faulting StreamID and Address.

## 3.2 Non-Functional Requirements

### 3.2.1 Quality Requirements

| Attribute | Description |
| :--- | :--- |
| **Feature** | N/A |
| **Quality Attribute** | Performance |
| **Input/Stimulus** | Heavy Map/Unmap load |
| **Output/Response** | Efficient command batching. |
| **Pass/Fail Condition** | Throughput degradation < 5% vs. bypass. |
| **Verification Method** | Performance Measurement |
| **Verification Environment** | Real-time Kernel Setup |

**Detailed Requirements:**
*   **REQ-SMMU-NF-01:** The driver SHOULD use <span style="{{ site.code }}">CMD_SYNC</span> sparingly to batch multiple TLB invalidation commands.
*   **REQ-SMMU-NF-02:** The interrupt handler for the Event Queue MUST NOT block for excessive periods.

### 3.2.2 Constraints

| Attribute | Description |
| :--- | :--- |
| **Feature** | N/A |
| **Constraint** | Interface Compliance |
| **Description** | The driver MUST implement the standard Linux `iommu_ops` structure. |

**Detailed Requirements:**
*   **REQ-SMMU-IF-01:** The driver MUST register with the Linux IOMMU subsystem using `iommu_device_register`.

# 4 Annex

## 4.1 Annex A. Domain Model
<img src="/assets/images/documents/aspice/smmu/smmu-domain-model.png" alt="SMMU Domain Model" width="600">

## 4.2 Annex B. Quality Scenario
**Scenario: Context Fault Handling**
<img src="/assets/images/documents/aspice/smmu/smmu-quality-scenario.png" alt="SMMU Fault Handling Scenario" width="600">

## 4.3 Annex C. Quality Scenario Analysis
*Analysis of the fault handling latency and robustness.*

# 5 Terms and Abbreviations

| Term | Description |
| :--- | :--- |
| **SMMU** | System Memory Management Unit |
| **IOVA** | Input Output Virtual Address |
| **STE** | Stream Table Entry |
| **CD** | Context Descriptor |
| **TLB** | Translation Lookaside Buffer |
| **IORT** | IO Remapping Table (ACPI) |

# 6 References

| Document Name | Version |
| :--- | :--- |
| ARM SMMUv3 Architecture Specification | IHI 0070 |
| Linux Kernel Source Code | 6.12 |
