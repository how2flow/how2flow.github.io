---
permalink: /documents/trace32/t32-reloc/
title: Trace32 symbol relocation
excerpt: "How to relocate t32 symbol"
comments: false
toc: true
---

## Trace32 relocation

Trace32 Debugging Environment in this posting
```
The host PC environment is a Windows 11 (x86 PC).
The target board environment is an ARM-A U-Boot
```
<br>

Trace32 symbol relocation is primarily used when debugging U-Boot, as the bootloader itself performs code relocation.<br>
<br>

### U-Boot bootloader

U-Boot performs relocation to move itself from a limited initial memory (like SRAM or ROM) to a larger memory area (usually DRAM) after initialization.<br>

The main reasons are:<br>
<B>Limited Initial Memory</B>: SRAM or ROM is too small to hold the full U-Boot image.<br>
<B>DRAM Initialization</B>: DRAM is not available at early boot; it must be initialized first.<br>
<B>Code and Data Relocation</B>: After moving to DRAM, pointers, global variables, and function tables must be updated to work correctly.<br>
<B>Stable Runtime Environment</B>: Relocation allows proper use of BSS, stack, and global data in DRAM.<br>
<br>
In short, relocation ensures U-Boot can run reliably in a larger, fully initialized memory environment.<br>
<br>

### Load U-Boot elf

Please refer to the basic [debug posting](/documents/trace32/t32-debug-ap/) for instructions on how to load symbols.<br>
<br>

<img src="/assets/images/documents/trace32/t32reloc0.png" alt="t32reloc0" width="640" height="480"><br>
relocation view..<br>
<br>

### Relocation

You need to identify the relocation offset first.<br>
<br>
There are two cases:<br>
An environment where the U-Boot console is available.<br>
An environment where the console is not available.<br>
<br>

#### Opt1. In U-Boot console

Use <span style="{{ site.code }}">bdinfo</span> In console.
```
=> bdinfo
boot_params = 0x0000000000000000
DRAM bank   = 0x0000000000000000
-> start    = 0x0000000020000000
-> size     = 0x00000000a0000000
DRAM bank   = 0x0000000000000001
-> start    = 0x00000001a0000000
-> size     = 0x0000000160000000
flashstart  = 0x0000000000000000
flashsize   = 0x0000000000000000
flashoffset = 0x0000000000000000
baudrate    = 115200 bps
relocaddr   = 0x0000000037f42000
reloc off   = 0x0000000007f42000
Build       = 64-bit
current eth = unknown
eth-1addr   = (not set)
IP addr     = 192.168.0.99
fdt_blob    = 0x0000000031f393a0
lmb_dump_all:
 memory.count = 0x2
 memory[0]      [0x20000000-0xbfffffff], 0xa0000000 bytes, flags: none
 memory[1]      [0x1a0000000-0x2ffffffff], 0x160000000 bytes, flags: none
 reserved.count = 0x2
 reserved[0]    [0x30f39390-0xbfffffff], 0x8f0c6c70 bytes, flags: no-overwrite
 reserved[1]    [0x1a0000000-0x2ffffffff], 0x160000000 bytes, flags: no-overwrite
devicetree  = separate
arch_number = 0x0000000000000000
TLB addr    = 0x0000000037ff0000
irq_sp      = 0x0000000031f39390
sp start    = 0x0000000031f39390
Early malloc usage: 1880 / 2000
```
<br>

In this case, reloc offset is <span style="{{ site.code }}">0x7f42000</span> .<br>
then, run this command in Trace32 PowerView
```
B:: sYmbol.RELOCate.shift 0x7f42000
```
<br>

#### Opt2. No U-Boot console

If the console is not available,<br>
you need to find the relocation offset using the ELF symbol information.<br>
Information related to relocation is stored in the gd(Global Data) symbol.

<img src="/assets/images/documents/trace32/t32reloc1.png" alt="t32reloc1" width="640" height="480"><br>

In this case, reloc offset is <span style="{{ site.code }}">0x7f42000</span> .<br>
then, run this command in Trace32 PowerView
```
B:: sYmbol.RELOCate.shift 0x7f42000
```
<br>

### Result

<B>Before relocation</B>:
<img src="/assets/images/documents/trace32/t32reloc2.png" alt="t32reloc2" width="640" height="480"><br>
<br>

<B>After relocation</B>:
<img src="/assets/images/documents/trace32/t32reloc3.png" alt="t32reloc3" width="640" height="480"><br>
<br>

## Appendix

### Relocation offset

Once the relocation address is determined,<br>
subtracting the code's entry point from it gives the relocation offset.
```
$ readelf -h u-boot
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0x30000000
  Start of program headers:          64 (bytes into file)
  Start of section headers:          6399408 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         2
  Size of section headers:           64 (bytes)
  Number of section headers:         22
  Section header string table index: 21
```
<br>

Relocation address is <span style="{{ site.code }}">0x37f42000</span> .<br>
Entry point is <span style="{{ site.code }}">0x30000000</span> .<br>
So, reloc offset is <span style="{{ site.code }}">0x7f42000</span> .<br>
<br>
