---
categories:
- linux
excerpt: Information mapping and utilization in U-BOOT to support multiple chipsets
  with a single driver.
header:
  teaser: /assets/images/posts/thumbnails/note.jpg
layout: single
tags:
- device-driver
- u-boot
title: '[Linux] Utilizing Driver ID Information and DTS'
toc: true
redirect_from:
- /documents/linux/u-boot-udevice/
- /legacy/u-boot-udevice/
- /documents/linux/u-boot-udevice
- /legacy/u-boot-udevice
---
## U-BOOT Device Driver

As of this writing, the structure of modern U-BOOT is similar to that of the Linux kernel.<br>
To cover various chips, the evolution of drivers<br>
must also proceed in a direction that supports multiple chipsets.<br>

To develop a minimal driver that applies a single interface<br>
across diverse environments (SoCs),<br>
techniques for distinguishing between different SoCs are crucial.<br>

This is not unique to U-BOOT drivers.<br>

From the driver's perspective, the specific SoC currently in use is meaningless;<br>
it simply needs to parse specific information and call the appropriate functions.<br>

### Parsing the Device Tree

The Linux kernel introduced the device tree<br>
to efficiently manage the growing number of devices.<br>
U-BOOT has followed the kernel's lead.<br>

Commands are executed based strictly on the parsed information from device tree properties.<br>
Below are a few examples.<br>
(The kernel also has APIs that serve similar roles.)<br>

In U-BOOT, there is the DTS parsing API <span style="{{ site.code }}">dev_read_TYPE</span>.
```
...
    #address-cells = <1>;
    #size-cells = <1>;
    ...
    foo@71004000 {
        compatible = "steve,foo";
        regs = <0x71004000 0x4>;
        clock = <&clk_peri PERI_TCT>;
        clock-frequency = "1000000";
    };
```
```
uint32_t reg;
uint32_t freq;

reg = dev_read_addr_index(dev, 0);
freq = dev_read_u32(dev, "clock-frequency");
```
In this case, 0x71004000 is stored in reg, and 1000000 is stored in freq.<br>
<br>

### Parsing the udevice ID

A mapping table is required to distinguish devices.<br>
In U-BOOT, the <span style="{{ site.code }}">udevice_id</span> type is widely used.<br>
In the mapping table, devices are primarily distinguished by compatible and data.
```
struct udevice_id {
    const char *compatible;
    ulong data;
};
```
```
static const struct udevice_id bar_ids[] = {
    { .compatible = "steve,foo", .data = FOO_DATA },
    { }
};

U_BOOT_DRIVER(bar) = {
    .name       = "bar",
    .id     = UCLASS_BAR,
    .of_match   = bar_ids,
    .ops        = &bar_ops,
    .probe      = bar_probe,
    .of_to_plat = bar_ofdata_to_platdata,
};
```
The compatible property distinguishes the driver,<br>
while the data property can distinguish behaviors within the same driver.<br>
<br>

To utilize the data property, you can use <span style="{{ site.code }}">dev_get_driver_data(dev)</span><br>
within the function connected to <span style="{{ site.code }}">of_to_plat</span>.<br>
```
struct foo_dev {
    int data;
};

static int bar_ofdata_to_platdata(udevice dev)
{
    struct foo_dev *foo = dev_get_priv(dev);
    ...
    foo->data = dev_get_driver_data(dev);
}
```
<span style="{{ site.code }}">dev_get_priv</span> is an API that retrieves the private information of the udevice.<br>
+ Getting the driver_data during the driver <span style="{{ site.code }}">probe</span> phase is also perfectly fine.<br>
<br>
