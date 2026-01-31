---
categories:
- linux
excerpt: How to create and modify Initramfs for Linux.
header:
  teaser: /assets/images/posts/thumbnails/note.jpg
layout: single
tags:
- boot
- cpio
- initramfs
- mkimage
title: '[Linux] Initramfs Guide'
toc: true
redirect_from:
- /documents/linux/initramfs/
- /legacy/initramfs/
- /documents/linux/initramfs
- /legacy/initramfs
---
## Introduction

To boot a Linux OS on an ARMv8 architecture CPU, at least three components must be present:
1. A bootloader to initiate the kernel (often U-Boot).
2. The Linux kernel itself.
3. A root file system (RootFS).

**Initramfs** (Initial RAM File System) acts as a bridge before the final file system is mounted. It is a RAM-based file system that identifies and mounts essential devices. Once the basic configuration is complete, it performs a `pivot_root` to transition to the main file system, such as Ubuntu.

For a method to boot without Initramfs, refer to:
[Booting Linux without Initramfs](/posts/tech/2024-01-08-boot-noinitrd/)
*(Note: This is a legacy method used before Initramfs became standard.)*

In contrast, the modern approach involves creating a file system tree, adding an init script, and packaging it using the `mkimage` command. Ensure that `CONFIG_BLK_DEV_INITRD` is enabled in your kernel configuration:
```
CONFIG_BLK_DEV_INITRD=y
```

## mkimage

`mkimage` is part of the `u-boot-tools` package. It is used to wrap the file system image with a header that U-Boot understands.

### Creating Initramfs

1. Prepare your root directory structure.
2. Create an `init` script.
3. Package it using `cpio` and `gzip`.
4. Wrap it with `mkimage`.

```bash
# Example command
find . | cpio -H newc -o | gzip -9 > ../initramfs.cpio.gz
mkimage -A arm64 -O linux -T ramdisk -d ../initramfs.cpio.gz ../initramfs.uimg
```
