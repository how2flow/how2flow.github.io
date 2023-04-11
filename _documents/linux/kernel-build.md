---
permalink: /documents/linux/kernel-build/
title: "Linux kernel build and install manual"
excerpt: "how to build linux kernel? Is there an example? (yes!)"
toc: true
---

## Linux kernel

The Linux kernel is a free and open-source, monolithic, modular, multitasking, Unix-like operating system kernel.<br>

Linux commands are of the Debian family.<br>
Linux version is <span style="{{ site.code }}">5.15 or later</span>

### How to build linux kernel

Download requirements
```
# apt-get install build-essential bc bison flex libssl-dev # or yum
```

Download source code
```
# git clone --depth=1 -b v5.15 https://github.com/torvalds/linux.git
```
When you download only the version you want, you only need to modify the <span style="{{ site.code }}">5.15</span> part.<br>
The reason for downloading this is that the linux source has a very large capacity. It's to save time.<br><br>

Kernel build
```
# cd linux
# make ARCH="$(arch)" defconfig
# make -j"$(nproc)"
```
When you load architectural information, the <span style="{{ site.code }}">arch</span> value<br>
and the Linux target name might not match.<br>
Then, run <span style="{{ site.code }}">make ARCH=$(dpkg --print-architecture) defconfig</span><br>
Optional, you can add make option <span style="{{ site.code }}">O=out</span>, If so, the rest of the processes must run <span style="{{ site.code }}">cd out</span> and proceed.<br><br>

#### cross-compile

If the architecture of the build machine and the target is different, try cross-compile.<br>

Download source code
```
# git clone --depth=1 -b v5.15 https://github.com/torvalds/linux.git
```
When you download only the version you want, you only need to modify the <span style="{{ site.code }}">5.15</span> part.<br>
The reason for downloading this is that the linux source has a very large capacity. It's to save time.<br><br>

Kernel cross-compile build
```
# cd linux
# make ARCH="your_target_archtecture" defconfig
# make CROSS_COMPILE="your_target_toolchain" -j"$(nproc)"
```

| arch | toolchain |
| :---: | :---: |
| arm | arm-linux-gnueabi- /<br> arm-linux-gnueabihf- |
| arm64 | aarch64-linux-gnu- |
| x86 | x86_64-linux-gnu- |
| etc | ... |

Optional, you can add make option <span style="{{ site.code }}">O=out</span>, If so, the rest of the processes must run <span style="{{ site.code }}">cd out</span> and proceed.<br><br>

### How to install linux kernel

If you did nothing and followed well, <span style="{{ site.code }}">pwd -P</span> is <span style="{{ site.code }}">$prefix/linux</span> . Most likely <span style="{{ site.code }}">~/linux</span> .<br>
If you had cross-compiled, mount the target's file system.<br>
And you should add option <span style="{{ site.code }}">INSTALL_DTBS_PATH="target's rootfs"</span> or <span style="{{ site.code }}">INSTALL_MOD_PATH="target's rootfs"</span> depends on installation type. (except image)<br>
There are three main types of installation.<br><br>

Kernel install: device-tree-blob(overlays)
```
# make dtbs_install
```

Kernel install: kernel-modules
```
# make modules_install
```

Kernel install: kernel-image (built-in)
```
# cp arch/${ARCH}/boot/Image.gz /boot/vmlinuz-$(uname -r) # target's vmlinuz
```

### Example

There is a [Linux kernel build & install example](/posts/kernel-build).<br>
