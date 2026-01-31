---
categories:
- linux
excerpt: A comprehensive guide on porting RPMsg for IPC between Linux and RTOS.
header:
  teaser: /assets/images/posts/thumbnails/note.jpg
layout: single
tags:
- openamp
- rpmsg
- zephyr
- embedded
- linux
title: '[Linux] RPMsg Porting Guide'
toc: true
redirect_from:
- /documents/linux/generic-msi/
- /legacy/generic-msi/
- /documents/linux/generic-msi
- /legacy/generic-msi
---
## Introduction

**RPMsg (Remote Processor Messaging)** is a standard protocol used to implement lightweight and efficient Inter-Processor Communication (IPC) in multi-core environments. It is widely utilized in systems with a Linux + MCU (or RTOS) architecture, making message exchange between heterogeneous processors simple and reliable.

### RPMsg Basic Structure

RPMsg operates on top of the **OpenAMP** framework. Its core components include:

#### Virtio
An interface that abstracts virtual devices.

#### Vring
A ring-buffered queue structure for actual data buffers. It manages incoming and outgoing messages based on these queues.

#### Remoteproc
A framework to load, boot, and manage remote processors. It primarily provides three functions: `start`, `stop`, and `kick`. While `start`/`stop` control the lifecycle of the remote device, `kick` acts as a doorbell (notification) for communication. Implementing a platform driver for specific vendor scenarios is crucial on the Master side.

#### RPMsg Endpoint
The actual channel through which messages are sent and received. Endpoints with the same name on both Linux and RTOS are connected to each other.

### Why Use RPMsg?
- Provides standard APIs, eliminating the need to implement custom IPC from scratch.
- Enables fast transmission based on shared memory structures.
- Supports major operating systems, including Linux and Zephyr RTOS.

## Porting Note

This porting scenario involves communication between two clusters using RPMsg. In this setup, the **Master** device typically runs Linux, and the **Remote** device runs an RTOS like Zephyr.

### Environment
- **Chip:** TCC807x
- **Master (Linux):** Kernel 5.10 (Sub-Core)
- **Remote (Zephyr):** RTOS 3.7 (Main Core)

The initial phase focuses on a **self-boot** scenario where the Remote device is already running before Linux boots. We implement only the `kick` function for doorbell interrupts and provide static **carveouts** (reserved memory regions). Since we use shared memory based on Virtio, **dcache coherency** (cache invalidate/flush) must be strictly guaranteed.

Since the `resource_table` is determined by the final image of the Remote device, we start the work from the Remote side.

### Remote Side: Zephyr Configuration

Zephyr provides built-in OpenAMP library support and samples. The `openamp_rsc_table` sample is an excellent starting point for Linux communication.

#### Cache Coherency
Ensure the following configurations are enabled in Zephyr to handle cache operations correctly:
```
CONFIG_CACHE_MANAGEMENT=y
CONFIG_OPENAMP_WITH_DCACHE=y
```
Verify these settings in `build/zephyr/.config` after running `west build`.

#### Self-boot Scenario Adjustments
Most drivers are designed for auto-boot. For a self-boot scenario, the `resource_table.h` in the OpenAMP library needs modification to use static addresses for Vrings:

```c
/* Before: Allocated by Master */
#define VRING_RX_ADDRESS -1
#define VRING_TX_ADDRESS -1
#define VRING_BUFF_ADDRESS -1

/* After: Static addresses for Self-boot */
#define VRING_RX_ADDRESS 0x200c9000UL
#define VRING_TX_ADDRESS 0x200c8000UL
#define VRING_BUFF_ADDRESS 0x200ca000UL
```

### Master Side: Linux Driver

Linux provides the RPMsg and Remoteproc frameworks out of the box in `drivers/rpmsg` and `drivers/remoteproc`. However, since our TCC807x scenario uses a self-booting remote processor, we develop a custom Remoteproc platform driver.

#### Execution Flow
1. **Probe:** The driver matches the `compatible` string in the Device Tree.
2. **rproc_alloc:** Creates a handle for the remote processor. Maps the `resource_table` from the reserved memory region.
3. **rproc_add_carveout:** Informs the framework about pre-allocated shared memory regions.
4. **rproc_boot:** In self-boot mode, it skips firmware loading and transitions the state to `RPROC_DETACHED`.
5. **Handle VDEV:** Processes `RSC_VDEV` entries in the resource table to define Virtio devices.
6. **Virtio Bus Registration:** Matches the `VIRTIO_ID_RPMSG` and triggers `rpmsg_probe`.
7. **Virtqueue Initialization:** Sets up RX/TX virtqueues based on the mapped Vring memory.
8. **Buffer Allocation:** Allocates actual message payloads and registers them to the receive virtqueue.

### Shared Memory Configuration (Device Tree)

The shared memory regions should be defined carefully in the Device Tree to match the Remote device's resource table layout.

```dts
&reserved_memory {
    rsc_table0: rsc-table@200c6000 {
        no-map;
        reg = <0x0 0x200c6000 0x0 0x2000>;
        tcc,rsc-offset = <0x3b8>;
        tcc,rsc-size = <0x1000>;
    };
    /* ... define Vrings and Buffers ... */
};
```

## Conclusion

After porting, the `rpmsg-client-sample` from the Linux kernel successfully communicated with the Zephyr remote. Although it took about three weeks to study Virtio/RPMsg and complete the porting, the result provides a robust foundation for multi-core communication.

<img src="/assets/images/posts/contents/linux/rpmsg-linux.png" alt="rpmsg-linux" width="320" height="240">
<img src="/assets/images/posts/contents/linux/rpmsg-zephyr.png" alt="rpmsg-zephyr" width="320" height="240">
