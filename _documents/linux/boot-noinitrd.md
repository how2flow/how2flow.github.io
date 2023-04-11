---
permalink: /documents/linux/boot-linux-noinitrd/
title: Boot Linux on Embedded board (arm64)
excerpt: "How to boot linux with uboot on target board arm64"
toc: true
---

## Environment

<span style="{{ site.code }}">Host</span> : Linux (debian type linux / Ubuntu 22.04LTS)<br>
<span style="{{ site.code }}">Storage</span> : local (SD/eMMC). (Use SD card in this post)<br>
<span style="{{ site.code }}">Bootloader</span> : U-boot<br>
<span style="{{ site.code }}">Kernel</span> : Linux kernel 5.10.y<br>
<span style="{{ site.code }}">Root-file-system</span> : https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-riscv64.squashfs<br>

### Warning (Must-Read)

It just provided the most common flow.<br>
Depending on the target, several courses can be changed or added.<br>
For more information, see Target's Guide<br>

## Prepare (No initrd)

The initrd image used to mount a temporary root file system using RAM like a disk<br>
to install the required drivers, but this is <span style="{{ site.code }}">optional</span> .<br>

By default, posts how to load the kernel and run the root file system without <span style="{{ site.code }}">initrd</span> .<br>

```
1. bootloader

2. kernel bootloader -> kernel load

3. mount temp root file system (ram or other mem)

4. mount real root file system (mount 'root=')

5. pivot_root & free temp root file system

6. init process in real root filesystem (linux/init/main.c)
```

When the Linux kernel loads the root filesystem,<br>
Before loading the actual root file system, mount some of the necessary drivers in temporary space<br>
to proceed with the root file system's init process.<br>

### U-boot

Each manufacturer has a guide to build and flash the U-boot.<br>
Follow the guide well to prepare the images you need.<br>
e.g. <span style="{{ site.code }}">u-boot-spl.img</span> , <span style="{{ site.code }}">u-boot.img</span> ...<br>

Prepare images and use them to flash it to the last.<br>

### Kernel

Each manufacturer provides a kernel build guide.<br>
Prepare a kernel image according to the guide.<br>
The kernel image is created in <span style="{{ site.code }}">arch/${ARCH}/boot/</span> when built.<br>

But if there's something important,<br>
Because not use the initrd,<br>
You need to modify one of the defconfigs.
```
CONFIG_BLK_DEV_INITRD=n
```
<br>

### Root file System

Download Root file system image.<br>
You can use anything, but I use Ubuntu 22.04 rfs cloud img in this post.<br>
<span style="{{ site.code }}">https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-arm64.squashfs</span> <br>

## Flash (No initrd)

### Creating a Boot Device

Prepare the SD card to use as a device and the SD card reader to connect to the host.<br>
I will separate the kernel image and the root file system.<br>
You can use tools such as <span style="{{ site.code }}">fdisk</span> , <span style="{{ site.code }}">gdisk</span> , and <span style="{{ site.code }}">parted</span> .<br>
In this post, use <span style="{{ site.code }}">fdisk</span> . <br>

SD card is <span style="{{ site.code }}">/dev/sdf</span>
```
$ sudo fdisk /dev/sdf

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-15523839, default 2048): 2048
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-15523839, default 15523839): +256M

Created a new partition 1 of type 'Linux' and of size 256 MiB.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 2
First sector (526336-15523839, default 526336):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (526336-15523839, default 15523839): 

Created a new partition 2 of type 'Linux' and of size 7.2 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

```
```
$ sudo fdisk -l
...
Device     Boot  Start      End  Sectors  Size Id Type
/dev/sdf1         2048   526335   524288  256M 83 Linux
/dev/sdf2       526336 15523839 14997504  7.2G 83 Linux
```

### Install kernel & root file system

Build kernel with <span style="{{ site.code }}">CONFIG_BLK_DEV_INITRD=n</span> and install to the SD that just prepared.<br>

Install modules with the make option <span style="{{ site.code }}">modules_install</span> and<br>
the parameter <span style="{{ site.code }}">INSTALL_MOD_PATH</span> allows you to install kernel modules directly<br>
in the desired path.<br>

Then, make a directory for boot
```
$ cd ~
$ mkdir -p MAKE
$ mkdir -p MAKE/BOOT
$ mkdir -p MAKE/BOOT/dtbs
$ mkdir -p MAKE/rootfs
$ cp {your_fdt_binary} MAKE/BOOT/dtbs
$ cp {your_kernel_image} MAKE/rootfs
$ sudo mount -o bind MAKE/BOOT MAKE/rootfs/boot
```
<br>

Install Root file system that just prepared.
```
$ cd ~
$ wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-arm64.squashfs
$ sudo unsquashfs -f -d MAKE/rootfs jammy-server-cloudimg-arm64.squashfs
```
<br>

File Tree
```
$ tree MAKE -L 2
MAKE
├── BOOT
│   ├── dtbs
│   └── {your_kernel_image}
└── rootfs
    ├── bin -> usr/bin
    ├── boot
    ├── dev
    ├── etc
    ├── home
    ├── lib -> usr/lib
    ├── media
    ├── mnt
    ├── opt
    ├── proc
    ├── root
    ├── run
    ├── sbin -> usr/sbin
    ├── snap
    ├── srv
    ├── sys
    ├── tmp
    ├── usr
    └── var
```
<br>

You need permission to access the file system.<br>
Permission should be obtained with a password,<br>
but the default image has a root account and no password,<br>
so you need to create a password.<br>

Following is the process of adding the root account password<br>
to the target's root file system.
```
$ sudo apt install qemu-user-static
$ sudo cp /usr/bin/qemu-aarch64-static ~/MAKE/rootfs/usr/bin
$ sudo chroot ~/MAKE/rootfs /usr/bin/qemu-aarch64-static /bin/sh -c 'echo -e "root\nroot\n" | passwd root'
```
<br>

id is <span style="{{ site.code }}">root</span> and password is <span style="{{ site.code }}">root</span> .<br>


Make device (uuid is optional)
```
$ uuidgen
7a50a940-6e36-4e52-b086-4be5d2fc4a48
$ sudo mkfs.ext2 -L BOOT -U 7a50a940-6e36-4e52-b086-4be5d2fc4a48 -d ~/MAKE/BOOT /dev/sdf1
$ uuidgen
2005bb8b-7383-4e7d-aa41-e10aa5e324ef
$ sudo mkfs.ext4 -L BOOT -U 7a50a940-6e36-4e52-b086-4be5d2fc4a48 -d ~/MAKE/rootfs /dev/sdf2
```
<br>

Flash U-boot to SD card
```
$ sudo dd if={uboot_spl_image} of=/dev/sdf conv=fsync,notrunc seek={spl_sector}
$ sudo dd if={uboot_image} of=/dev/sdf conv=fsync,notrunc seek={uboot_sector}
```
```
$ sync
$ sudo eject /dev/sdf
```
<br>


## Boot System (no initrd)

Turn on the target and Enter to the U-boot prompt with <span style="{{ site.code }}">serial console</span> .<br>
The console can be specified in <span style="{{ site.code }}">bootargs</span> , and it is usually related to the target's guidance.<br>
Suppose the console is <span style="{{ site.code }}">ttyS1</span> and that it is connected to the host.<br>
The baud rate is defined in the kernel.<br>

Prepare the prompt using the <span style="{{ site.code }}">minicom</span> .
```
$ sudo minicom -D {device_connected_to_target's_ttyS1} -b {target's_baudrate}
```
<br>

### U-boot prompt

Please consider that each target has a different U-boot configuration.<br>
<br>

#### Check mmc dev
```
=> mmc dev 0
Card did not respond to voltage select!
=> mmc dev 1
switch to partitions #0, OK
mmc1(part0) is current device
```
The device we used is mmc1.<br>
The first partition is set to 0,<br>
the second partition is set to 1, and the index is set.<br>
<br>

#### Bootargs
```
=> echo ${bootargs}
storagemedia=sd console=ttyS1
```

You must specify the path on which the actual root file system will be mounted.<br>
We assume that this is the second partition of device 1<br>
and the target's valid driver is mmcblk (depending on target guidance)
```
=> setenv bootargs "${bootargs} root=/dev/mmcblk1p2 rw rootwait
```

Or, You can also use uuid.<br>
In the previous step, specified uuid.
```
=> setenv bootargs "${bootargs} root=UUID=2005bb8b-7383-4e7d-aa41-e10aa5e324ef rw rootwait
```
<span style="{{ site.code }}">rootwait</span> : Use it because it takes time for the device to actually mount.<br>
<br>

Now, finally, you can load and boot the fdt file and kernel.<br>
The address of the load is usually in guidance, or if not, select a random address.<br>
If you use an address that is unavailable, a message is displayed.<br>
<br>

#### Load kernel
```
=> load mmc 1:2 {load_fdt_address} /boot/dtbs/{your_fdt_binary} 
239134 bytes read in 42 ms (5.4 MiB/s)
=> load mmc 1:2 {load_kernel_address} /boot/{your_kernel_image}
36178432 bytes read in 9370 ms (3.7 MiB/s)
```
```
=> booti {load_kernel_address} - {load_fdt_address}
```
<br>

When you build the kernel, the images come in two types.<br>
Image binaries and compressed files appear.<br>
If you used a compressed kernel image to save capacity, you must load the kernel image through the following commands.
```
=> load mmc 1:2 {load_fdt_address} /boot/dtbs/{your_fdt_binary} 
239134 bytes read in 42 ms (5.4 MiB/s)
=> load mmc 1:2 {temp_kernel_address} /boot/{your_compressed_kernel_image}
36178432 bytes read in 9370 ms (3.7 MiB/s)
=> unzip {temp_kernel_address} {load_kernel_address}
```
```
=> booti {load_kernel_address} - {load_fdt_address}
```
<br>

### OS

Login ubuntu
```
login: root
password: root
```
```
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.10.198-odroid-arm64 aarch64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Fri Mar 29 03:41:37 UTC 2024

  System load:  0.70166015625     Temperature:           34.2 C
  Usage of /:   20.1% of 6.94GB   Processes:             234
  Memory usage: 2%                Users logged in:       0
  Swap usage:   0%                IPv4 address for eth0: 192.168.11.3


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


Last login: Thu Mar 28 08:59:00 UTC 2024 from 192.168.11.2 on pts/0
```
```
root@ubuntu:~$
```
<br>

If the network is not caught,<br>
you can check the kernel configuration<br>
or you can check the configuration of <span style="{{ site.code }}">/etc/netplan</span> .<br>
try add <span style="{{ site.code }}">renderer: networkd</span> .<br>
