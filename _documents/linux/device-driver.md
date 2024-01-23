---
permalink: /documents/linux/device-driver/
title: "Device driver"
excerpt: "how to flow device-dirver. device, drivers, device driver, device node ..."
toc: true
---

## info

A Linux device driver is software that is responsible for communication between the Linux kernel and hardware.<br>
It provides an interface between <span style="{{ site.code }}">kernel space</span> and <span style="{{ site.code }}">user space</span> ,<br>
and it interacts with specific hardware devices to make them available to the operating system.<br>

There are three main types of device drivers.<br>

### char device driver

Deals with sequentiality of data.<br>

### block device driver

Managed by the file system inside the kernel and with internal buffers.<br>

### network device driver

Provides the ability to connect with the network layer to send and receive network packets over network communication.<br>

## device drivers

How device drivers work and how they are created/added.<br>
Device drivers are software for using hardware in a kernel or user space, as described above.<br>

### intro

This is a graph that is how a device is being used in linux system.
```
|  User Space  |                Kernel Space                 |              Hardware Space               |
.              .                                             .                                           .
.              .                                             .                                           .
.               \                                            .                                +----------+
.              +-+                              +------------+      +------------+            |          |
+------------+ | |                              |            |      |            | <--------> | Device 1 |
| user app 1 | | |-> System call/Device call -> |            |      |            |            |          |
+------------+ | |                              |            |      |            |            +----------+
               | |                              |            |      |            |            +----------+
+------------+ | |                              |            | <--> |   Device   |  hardware  |          |
| user app 2 | | |-> System call/Device call -> | Bus Driver | <--> |            |  protocol  | Device 1 |
+------------+ | |                              |            | <--> | Controller |            |          |
               | |                              |            |      |            |            +----------+
+------------+ | |                              |            |      |            |            +----------+
| user app 3 | | |-> System call/Device call -> |            |      |            |            |          |
+------------+ | |                              |            |      |            | <--------> | Device 1 |
.              +-+                              +------------+      +------------+            |          |
.     ...       / ` System call                              .                                +----------+
.              .     interface                               .                                           .
.              .                                             .                                           .
|  User Space  |                Kernel Space                 |              Hardware Space               |

```
<br>

Device drivers exist in kernel space.<br>
It takes system calls from the user space and connects the bus driver & the device controller.<br>
Or, it means that the bus driver is included.<br>

### file (node)

UNIX-based device drivers are in file format.<br>
Typically present in the <span style="{{ site.code }}">/dev</span> path.<br>
Device files can be created with the <span style="{{ site.code }}">mknod</span> command.
```
$ mknod /dev/{name} {type} {major} {minor}
```
<br>

type has block, character and FIFO device, see <span style="{{ site.code }}">mknod --help</span> for more information.<br>
As you can see from the command format, the device drivers have major, minor numbers,<br>
which are mentioned again in the driver code making section.<br>

### udev

It is Daemon that created for the purpose of automatically generating device files.<br>
Dynamic Device File Allocation.<br>
Provide API to gather information from system devices (sysfs)<br>

**udev-rules**<br>
<span style="{{ site.code }}">uevent</span> occurs when the device is connected,<br>
and the udev daemon automatically creates the device driver.<br>

You can add drivers to dynamically assign by creating a rules file.<br>
The path is <span style="{{ site.code }}">/etc/udev/rules.d</span> . and rules name is <span style="{{ site.code }}">##-name.rules</span> , <br>
The preceding number refers to the order in which the rules file is called.<br>

for example,<br>
50-uart.rules 70-canfd.rules When there are two files, uart rules apply first.<br>

### code generate

The part of the device driver implementation.<br>
Device drivers are strictly part of the kernel;<br>
you must follow the kernel's policy to create.<br>

In the kernel, device drivers are primarily managed on a kernel module(.ko) basis.<br>
However, there are drivers that only apply to certain architectures,<br>
called <span style="{{ site.code }}">platform device drivers</span> .<br>

It is very important to know this.<br>
Because all the models with Linux kernel are very various architectures.<br>
**<U>One driver can't be compatible with all architectures</U>**.<br>

This is because each chip has a different device controller.<br>
In fact, most drivers that you add(will add) are <span style="{{ site.code }}">platform device drivers</span> .<br>

Above all, this chapter takes <span style="{{ site.code }}">platform device drivers</span> as examples.<br>

#### license

The license for the kernel is a <span style="{{ site.code }}">GPL</span> license.<br>
The same goes for device drivers.<br>
Any code that operates within the kernel should be published as an open source; this is not limited to device drivers.<br>

The code first writes the license and copyright syntax.
```
// SPDX-License-Identifier: GPL-2.0-only
/*
 * Copyright (c) year name <e-mail>
 */
```

#### file_operation

The File Operation defines all actions associated with a file.<br>
Required for device driver initialization.<br>

The following must be present in the driver code.
```
static const struct file_operations
driver_name_fops = {
	.owner = THIS_MODULE
	.open = /* pointer func */
	.release = /* pointer func */
	.read = /* pointer func */
	.write = /* pointer func */
	.mmap = /* pointer func */

	...
};
```
<br>

You define the variables and functions required for the configuration.<br>
<span style="{{ site.code }}">owner</span> , <span style="{{ site.code }}">open</span> , <span style="{{ site.code }}">release</span> must be defined at the very least.<br>


#### driver definition

This is that you define and add drivers.<br>
It is essential for driver components and must be defined together, such as <span style="{{ site.code }}">file_operation</span> .<br>

The following must be present in the driver code.
```
static struct platform_driver driver_name_driver = {
	.probe = /* pointer func */
	.remove = /* pointer func */
	.driver = {
			.name = /* vars */
			.owner = THIS_MODULE,
			.of_match_table = /* pointer const */
			...
	};
	...
};
```
<br>

You define the variables and functions required for the configuration.<br>
<span style="{{ site.code }}">probe</span> , <span style="{{ site.code }}">remove</span> , <span style="{{ site.code }}">driver</span> must be defined at the very least.<br>

And we have to keep an eye on the <span style="{{ site.code }}">of_match_table</span> .<br>
The match table must be added as follows, and a macro is used for the addition.
```
static const struct of_device_id driver_name_of_match[] = {
	{.compatible = "{vendor},{driver}",},
};

...

MODULE_DEVICE_TABLE(of, driver_name_of_match);
```
<br>

The part of the Linux kernel that allows device drivers to be managed as a device tree.<br>
Through <span style="{{ site.code }}">MODULE_DEVICE_TABLE</span>, registering a match table allows you to manage drivers against the compatible properties of the device tree during kernel booting.<br>
This section is related to the Device Driver Management section of the kernel,<br>
See the [Device Tree section](/documents/linux/device-tree/).<br>
<span style="{{ site.code }}">\<linux/of.h\></span> must be included.<br>
<br>

Register the device driver with the following macro.
```
module_platform_driver(driver_name_driver);
```
<br>

Finally, create a macro related to this driver's information.
```
MODULE_ALIAS("platform:driver_name");
MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("...");
MODULE_AUTHOR("name <e-mail>");
```
<br>

#### make

Add driver-related content from <span style="{{ site.code }}">Makefile</span> that exists in the same path of the driver you added.<br>
In some cases, you may need to add or modify the <span style="{{ site.code }}">Kconfig</span> .<br>

