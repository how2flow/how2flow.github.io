---
permalink: /documents/linux/device-driver-examples/
title: "Device driver examples"
excerpt: "device driver examples"
toc: true
---

## Case 1: New device driver

This is How to add new device driver.<br>
If you add a new driver, you need a new configuration and source for the new driver.
```
Steps
1. Add new config
2. Add new driver code
3. Build kernel
```

### add new driver and build kernel

config files is in <span style="{{ site.code }}">arch/${ARCH}/configs/"target_config"</span> .
```
...
CONFIG_NEW_DEVICE_DRIVER=y
...
```
<br>

The definition for config about the driver should be added to the <span style="{{ site.code }}">Kconfig</span> that exists in the same path as the <span style="{{ site.code }}">Makefile</span> to build.
```
# Kconfig
...
config NEW_DEVICE_DRIVER
	tristate "USER DEBUG"
	default m
	help
	  user debug driver.
...
```
```
# Makefile
...
+obj-$(CONFIG_NEW_DEVICE_DRIVER) += new_device_driver.o
```

You can add the driver code to where you need it and build the kernel.<br>

### example

Add drivers for debugging the kernel of rockchip.<br>

```
  commit 74882ebe953d5e7e0a79e43a96176eb1223c44d2
  Author: Steve Jeong <steve@how2flow.net>
  Date:   Tue Mar 14 17:53:10 2023 +0900

      ODROID-M1: drivers: Add user_debug

      Signed-off-by: Steve Jeong <steve@how2flow.net>
      Change-Id: I9f5f307e40fb25a85d5077231138000fab37f1c6

  diff --git a/arch/arm64/configs/odroidm1_defconfig b/arch/arm64/configs/odroidm1_defconfig
  index 07224521df6f..d8b79b3129b9 100644
  --- a/arch/arm64/configs/odroidm1_defconfig
  +++ b/arch/arm64/configs/odroidm1_defconfig
  @@ -5873,3 +5873,4 @@ CONFIG_STRICT_DEVMEM=y
   # CONFIG_DEBUG_ALIGN_RODATA is not set
   # CONFIG_ARM64_RELOC_TEST is not set
   # CONFIG_CORESIGHT is not set
  +CONFIG_USER_DEBUG=y
  diff --git a/drivers/soc/rockchip/Kconfig b/drivers/soc/rockchip/Kconfig
  index 2b071773ef8a..3ec77815b28f 100644
  --- a/drivers/soc/rockchip/Kconfig
  +++ b/drivers/soc/rockchip/Kconfig
  @@ -160,4 +160,10 @@ config ROCKCHIP_SCHED_PERFORMANCE_BIAS
          Say y here to enable rockchip optimize task scheduler for performance bias,
          this would cause a little more power consumption.

  +config USER_DEBUG
  +     tristate "USER DEBUG"
  +     default m
  +     help
  +      user debug driver.
  +
   endif
  diff --git a/drivers/soc/rockchip/Makefile b/drivers/soc/rockchip/Makefile
  index 0f18d2b12f81..844120dfcdb2 100644
  --- a/drivers/soc/rockchip/Makefile
  +++ b/drivers/soc/rockchip/Makefile
  @@ -23,3 +23,4 @@ obj-$(CONFIG_ROCKCHIP_THUNDER_BOOT_CRYPTO) += rockchip_thunderboot_crypto.o
   obj-$(CONFIG_ROCKCHIP_THUNDER_BOOT_MMC) += rockchip_thunderboot_mmc.o
   obj-$(CONFIG_ROCKCHIP_THUNDER_BOOT_SFC) += rockchip_thunderboot_sfc.o
   obj-$(CONFIG_ROCKCHIP_DEBUG) += rockchip_debug.o
  +obj-$(CONFIG_USER_DEBUG) += user_debug.o
  diff --git a/drivers/soc/rockchip/user_debug.c b/drivers/soc/rockchip/user_debug.c
  new file mode 100644
  index 000000000000..7be99ffcec4a
  --- /dev/null
  +++ b/drivers/soc/rockchip/user_debug.c
  +/* Copyright (c) 2023 Steve Jeong <steve@how2flow.net>
  + *
  + * This program is free software; you can redistribute it and/or modify
  + * it under the terms of the GNU General Public License version 2 as
  + * published by the Free Software Foundation.
  + *
  + * path: drivers/soc/${vendor}/user_debugfs.c
  + * written by steve.jeong
  + */
  +
  +#include <linux/module.h>
  +#include <linux/moduleparam.h>
  +#include <linux/platform_device.h>
  +#include <linux/io.h>
  +#include <linux/init.h>
  +#include <linux/memblock.h>
  +#include <linux/slab.h>
  +#include <linux/of.h>
  +#include <linux/of_address.h>
  +#include <linux/cpu.h>
  +#include <linux/delay.h>
  +#include <asm/setup.h>
  +#include <linux/input.h>
  +#include <linux/debugfs.h>
  +#include <linux/timer.h>
  +#include <linux/workqueue.h>
  +#include <linux/slub_def.h>
  +#include <linux/uaccess.h>
  +#include <asm/memory.h>
  +
  +uint32_t user_debug_state = 0x1000;
  +static struct dentry *user_kernel_debug_debugfs_root;
  +
  +static int user_kernel_debug_stat_get(void *data, u64 *val)
  +{
  +       printk("===[%s][L:%d][val:%d]===\n", __func__, __LINE__, user_debug_state);
  +       *val = user_debug_state;
  +
  +       return 0;
  +}
  +static int user_kernel_debug_stat_set(void *data, u64 val)
  +{
  +       user_debug_state = (uint32_t)val;
  +       printk("===[user] [%s][L:%d], user_debug_state[%lu], value[%lu]===\n",
  +                       __func__, __LINE__, (long unsigned int)user_debug_state, (long unsigned int)val);
  +
  +       return 0;
  +}
  +
  +DEFINE_SIMPLE_ATTRIBUTE(user_kernel_debug_stat_fops,
  +               user_kernel_debug_stat_get, user_kernel_debug_stat_set, "%llu\n");
  +
  +static int user_kernel_debug_debugfs_driver_probe(struct platform_device *pdev)
  +{
  +       printk("===[%s][L:%d]===\n", __func__, __LINE__);
  +
  +       return 0;
  +}
  +
  +static struct platform_driver user_kernel_debug_debugfs_driver = {
  +       .probe = user_kernel_debug_debugfs_driver_probe,
  +       .driver = {
  +               .owner = THIS_MODULE,
  +               .name = "user_debug",
  +       },
  +};
  +
  +static int __init user_kernel_debug_debugfs_init(void)
  +{
  +       printk("===[%s][L:%d]===\n", __func__, __LINE__);
  +
  +       user_kernel_debug_debugfs_root = debugfs_create_dir("user_debug", NULL);
  +       debugfs_create_file("state", S_IRUGO, user_kernel_debug_debugfs_root,
  +                       NULL, &user_kernel_debug_stat_fops);
  +
  +       return platform_driver_register(&user_kernel_debug_debugfs_driver);
  +}
  +
  +late_initcall(user_kernel_debug_debugfs_init);
  +
  +MODULE_DESCRIPTION("user debug interface driver");
  +MODULE_AUTHOR("steve.jeong <how2soft@gmail.com>");
  +MODULE_LICENSE("GPL");
```
I will post a separate description of the code.<br>

<br>
Add it to the kernel code you want.<br>
Log the dmesg to a particular driver whenever that function is called from a particular function.
```
extern uint32_t user_debug_state;
...
if (1 == user_debug_state) printk("steve %s %d\n", __func__, __LINE__);
```

## Case 2: New device's Attr

To test the behavior of a device by reading or writing data.<br>
<span style="{{ site.code }}">DEVICE_ATTR</span> is define an attribute of device.<br>

```
Steps:
1. make device attribute
2. create & remove file.
```

### make device attribute

device attribute format:<br>
<span style="{{ site.code }}">DEVICE_ATTR(_name, _flag, _show, _store)</span><br>

parameters:<br>
<span style="{{ site.code }}">_name</span> : device attribute name. it will attach a prefix(<span style="{{ site.code }}">dev_attr_</span>)<br>

<span style="{{ site.code }}">_flag</span> : node permission flag. ex) <span style="{{ site.code }}">644</span>, <span style="{{ site.code }}">755</span> ...<br>

<span style="{{ site.code }}">_show</span> : the function pointer to hand over values to userspace from kernel.<br>

<span style="{{ site.code }}">_store</span>: the function pointer to hand over values to kernel from userspace.<br>

```
static ssize_t foo_show(struct device *dev, struct device_attribute *attr, char *buf)
{
	int count;

	...

	return count;
}

static ssize_t foo_store(struct device *dev, struct device_attribute *attr, char *buf, size_t count)
{
	int data;

	...

	return count;
}

DEVICE_ATTR(bar, 644, foo_show, foo_store);

...

```

### create & remove file.

If an attribute is created, create file at <span style="{{ site.code }}">probe</span> and remove at <span style="{{ site.code }}">remove</span>.

```
static int xyz_probe(struct platform_device *pdev)
{
	...

	ret = device_create_file(&pdev->dev, &dev_attr_bar);
	if (ret < 0) {
		dev_err(&pdev->dev, "Failed to create speed attribute file: %d\n", ret);
	}

	...
}

static int xyz_remove(struct platform_device *pdev)
{
	...

	device_remove_file(&pdev->dev, &dev_attr_bar);

	...
}

...

```

### example
Let me give an example.<br>

<a href="https://forum.odroid.com/viewtopic.php?p=368607#p368607">How to change i2c speed and use i2c-1 port</a><br>

The situation was that someone wanted to change the speed of the i2c on the odroid-m1 board.<br>
At first, I was told me to solve it by entering a value in the **speed node** of i2c like N2/C4,<br>
but M1 did not have a **speed node.**<br>

So I suggested adding clock-frequency on the device-tree.<br>
But I tried to change the speed dynamically through **speed node** like N2 and C4.<br>

This is diff of i2c driver code.<br>
source: <a href="https://github.com/hardkernel/linux/blob/odroidm1-4.19.y/drivers/i2c/busses/i2c-rk3x.c">github</a>

```
commit d997a909258b4dc02b7def2b64de6e51f07e40fb
Author: Steve Jeong <how2soft@gmail.com>
Date:   Fri Apr 7 12:03:14 2023 +0900

    ODROID-M1: driver/i2c: Add driver "speed" attribution

    for change i2c bus freq dynamically.

    e.g.
      $ echo 400000 | sudo tee /sys/bus/i2c/devices/i2c-0/device/speed

    Signed-off-by: Steve Jeong <how2soft@gmail.com>
    Change-Id: Ifcccf9bb61ed65133b64c803b53fb4e46d470e26

diff --git a/drivers/i2c/busses/i2c-rk3x.c b/drivers/i2c/busses/i2c-rk3x.c
index 62715a318fde..be119745bae5 100644
--- a/drivers/i2c/busses/i2c-rk3x.c
+++ b/drivers/i2c/busses/i2c-rk3x.c
@@ -1288,6 +1288,32 @@ static const struct of_device_id rk3x_i2c_match[] = {
 };
 MODULE_DEVICE_TABLE(of, rk3x_i2c_match);

+static ssize_t rk3x_i2c_get_speed(struct device *dev, struct device_attribute *attr, char *buf)
+{
+       struct rk3x_i2c *i2c = dev_get_drvdata(dev);
+       struct i2c_timings *t = &i2c->t;
+
+       return snprintf(buf, PAGE_SIZE, "%u\n", t->bus_freq_hz);
+}
+
+static ssize_t rk3x_i2c_set_speed(struct device *dev, struct device_attribute *attr, const char *buf, size_t count)
+{
+       struct rk3x_i2c *i2c = dev_get_drvdata(dev);
+       struct i2c_timings *t = &i2c->t;
+       int ret, freq;
+
+       ret = kstrtoint(buf, 10, &freq);
+       if (ret < 0)
+               return ret;
+
+       t->bus_freq_hz = (uint32_t)freq;
+       rk3x_i2c_adapt_div(i2c, clk_get_rate(i2c->clk));
+
+       return count;
+}
+
+DEVICE_ATTR(speed, S_IRUGO | S_IWUSR, rk3x_i2c_get_speed, rk3x_i2c_set_speed);
+
 static int rk3x_i2c_probe(struct platform_device *pdev)
 {
        struct device_node *np = pdev->dev.of_node;
@@ -1303,6 +1329,11 @@ static int rk3x_i2c_probe(struct platform_device *pdev)
        if (!i2c)
                return -ENOMEM;
 
+       ret = device_create_file(&pdev->dev, &dev_attr_speed);
+       if (ret < 0) {
+               dev_err(&pdev->dev, "Failed to create speed attribute file: %d\n", ret);
+       }
+
        match = of_match_node(rk3x_i2c_match, np);
        i2c->soc_data = match->data;
 
@@ -1454,6 +1485,7 @@ static int rk3x_i2c_remove(struct platform_device *pdev)
 
        i2c_del_adapter(&i2c->adap);
 
+       device_remove_file(&pdev->dev, &dev_attr_speed);
        clk_notifier_unregister(i2c->clk, &i2c->clk_rate_nb);
        unregister_pre_restart_handler(&i2c->i2c_restart_nb);
        clk_unprepare(i2c->pclk);
```

## Case 3: New Kernel params

Boot param is in <span style="{{ site.code }}">/proc/cmdline</span><br>
I will use kernel macro <span style="{{ site.code }}">__setup</span> and <span style="{{ site.code }}">kstrtoul</span> to add new boot parameters.
```
Steps
1. Use __setup macro with function that retrun params
2. Export symbol that is call the function (step 1)
```

### make boot parameter

Use __setup macro<br>
The format is:
```
...
__setup("string=", _func);
```

<span style="{{ site.code }}">_func</span> returns a value by parsing a string containing "string=".<br>
Save the return value of <span style="{{ site.code }}">_func</span> as the value of the parameter.<br>

```
...
static unsigned long param;
int __init get_value(char *str) {
	int ret;

	mutex_lock(&target_mutex);
	ret = kstrtoul(str, 10, &param);
	mutex_unlock(&target_mutex);

	return ret;
}
unsigned log set_value(void)
{
	return param;
}
...
__setup("new-param=", get_value);
```

Mutex is just one example.<br>
Only <span style="{{ site.code }}">mutual exclusion</span> from scheduling is required during parameter parsing.<br>
If there is a function that reads a value, the function that writes the value always follows.<br>
<br>

Most kernel parameters are initialized in the init layer, and performance is often applied in the higher layer.<br>
Symbols must be exported so that functions can be used from external sources.<br>
```
...
EXPORT_SYMBOL_GPL(set_value);
```
<br>

You can now apply kernel parameters to a driver via the <span style="{{ site.code }}">set_value</span> on an external driver.<br>

### example
Let me give an example.<br>

This is an example of actually applying the canfd clock in the rockchip kernel by adding the "can-clk=" boot parameter.<br>

```
commit ca875e1a17fea8c10abf00edef57527a3014f333
Author: Steve Jeong <how2soft@gmail.com>
Date:   Tue May 30 17:41:45 2023 +0900

    driver/clk: Add can-clk boot param
 
    Signed-off-by: Steve Jeong <how2soft@gmail.com>
    Change-Id: Id865ad6db803ed590d6f3b4264f6fc29dc78c9fd

diff --git a/drivers/clk/clk.c b/drivers/clk/clk.c
index 455cf0e5dfcc..9e7934c402f5 100644
--- a/drivers/clk/clk.c
+++ b/drivers/clk/clk.c
@@ -1739,6 +1739,26 @@ static unsigned long clk_core_get_rate(struct clk_core *core)
        return rate;
 }
 
+static unsigned long bios_clk_freq;
+int __init get_bios_clk(char *str) {
+       int ret;
+
+       clk_prepare_lock();
+
+       ret = kstrtoul(str, 10, &bios_clk_freq);
+
+       clk_prepare_unlock();
+
+       return ret;
+}
+
+unsigned long set_bios_clk(void)
+{
+       return bios_clk_freq;
+}
+__setup("can-clk=", get_bios_clk);
+EXPORT_SYMBOL_GPL(set_bios_clk);
+
 /**
  * clk_get_rate - return the rate of clk 
  * @clk: the clk whose rate is being returned
@@ -4467,7 +4487,6 @@ struct clk *devm_clk_register(struct device *dev, struct clk_hw *hw)
        return clk;
 }
 EXPORT_SYMBOL_GPL(devm_clk_register);
-
 /**
  * devm_clk_hw_register - resource managed clk_hw_register()
  * @dev: device that is registering this clock
diff --git a/drivers/net/can/rockchip/rockchip_canfd.c b/drivers/net/can/rockchip/rockchip_canfd.c
index 69ca9cd9d634..0179ceceaf32 100644
--- a/drivers/net/can/rockchip/rockchip_canfd.c
+++ b/drivers/net/can/rockchip/rockchip_canfd.c
@@ -928,6 +928,7 @@ static int rockchip_canfd_probe(struct platform_device *pdev)
        struct resource *res;
        void __iomem *addr;
        int err, irq;
+       unsigned long bios_clk_rate;
 
        irq = platform_get_irq(pdev, 0);
        if (irq < 0) {
@@ -966,7 +967,11 @@ static int rockchip_canfd_probe(struct platform_device *pdev)
                return -ENODEV;
 
        rcan->base = addr;
-       rcan->can.clock.freq = clk_get_rate(rcan->clks[0].clk);
+       bios_clk_rate = set_bios_clk();
+       if(!bios_clk_rate)
+               rcan->can.clock.freq = clk_get_rate(rcan->clks[0].clk);
+       else
+               rcan->can.clock.freq = set_bios_clk();
        rcan->dev = &pdev->dev;
        rcan->can.state = CAN_STATE_STOPPED;
        rcan->can.bittiming_const = &rockchip_canfd_bittiming_const;
diff --git a/include/linux/clk.h b/include/linux/clk.h
index 9c0d0ff2a9f5..0d37f015b072 100644
--- a/include/linux/clk.h
+++ b/include/linux/clk.h
@@ -543,6 +543,9 @@ void clk_bulk_disable(int num_clks, const struct clk_bulk_data *clks);
  */
 unsigned long clk_get_rate(struct clk *clk);
 
+int __init get_bios_clk(char *str);
+unsigned long set_bios_clk(void);
+
 /**
  * clk_put     - "free" the clock source
  * @clk: clock source
```
