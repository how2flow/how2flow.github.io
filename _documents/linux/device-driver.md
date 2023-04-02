---
permalink: /documents/linux/device-driver/
title: "Device driver"
toc: true
---

## New Device Attribute

To test the behavior of a device by reading or writing data.<br>
`DEVICE_ATTR`is define an attribute of device.<br>

```
Steps:
1. make device attribute
2. create & remove file.
```

### make device attribute

device attribute format:<br>
`DEVICE_ATTR(_name, _flag, _show, _store)`<br>
<br>
parameters:<br>
`_name`: device attribute name. it will attach a prefix(`dev_attr_`)<br>
`_flag`: node permission flag. ex) `644`, `755` ...<br>
`_show`: the function pointer to hand over values to userspace from kernel.<br>
`_store`: the function pointer to hand over values to kernel from userspace.<br>

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

If an attribute is created, create file at `probe` and remove at `remove`.

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
<br>
<a href="https://forum.odroid.com/viewtopic.php?p=368607#p368607">How to change i2c speed and use i2c-1 port</a><br>
<br>
The situation was that someone wanted to change the speed of the i2c on the odroid-m1 board.<br>
At first, I was told me to solve it by entering a value in the **speed node** of i2c like N2/C4,<br>
but M1 did not have a **speed node.**<br>
<br>
So I suggested adding clock-frequency on the device-tree.<br>
But I tried to change the speed dynamically through **speed node** like N2 and C4.<br>

This is diff of i2c driver code.<br>
source: <a href="https://github.com/hardkernel/linux/blob/odroidm1-4.19.y/drivers/i2c/busses/i2c-rk3x.c">github</a>

```
commit 8d8b92a83016fdc9b135abe6245e382240657ffe
Author: Steve Jeong <how2soft@gmail.com>
Date:   Fri Apr 7 12:03:14 2023 +0900

    ODROID-M1: driver/i2c: Add driver "speed" attribution.
    
    for change i2c bus freq dynamically.
    
    e.g.
      $ echo "400000" > /sys/bus/i2c/devices/i2c-0/device/speed
    
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
+	struct rk3x_i2c *i2c = dev_get_drvdata(dev);
+	struct i2c_timings *t = &i2c->t;
+
+	return snprintf(buf, PAGE_SIZE, "%u\n", t->bus_freq_hz);
+}
+
+static ssize_t rk3x_i2c_set_speed(struct device *dev, struct device_attribute *attr, const char *buf, size_t count)
+{
+	struct rk3x_i2c *i2c = dev_get_drvdata(dev);
+	struct i2c_timings *t = &i2c->t;
+	int ret, freq;
+
+	ret = kstrtoint(buf, 10, &freq);
+	if (ret < 0)
+		return ret;
+
+	t->bus_freq_hz = (uint32_t)freq;
+	rk3x_i2c_adapt_div(i2c, clk_get_rate(i2c->clk));
+
+	return count;
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
 
+	ret = device_create_file(&pdev->dev, &dev_attr_speed);
+	if (ret < 0) {
+		dev_err(&pdev->dev, "Failed to create speed attribute file: %d\n", ret);
+	}
+
 	match = of_match_node(rk3x_i2c_match, np);
 	i2c->soc_data = match->data;
 
@@ -1454,6 +1485,7 @@ static int rk3x_i2c_remove(struct platform_device *pdev)
 
 	i2c_del_adapter(&i2c->adap);
 
+	device_remove_file(&pdev->dev, &dev_attr_speed);
 	clk_notifier_unregister(i2c->clk, &i2c->clk_rate_nb);
 	unregister_pre_restart_handler(&i2c->i2c_restart_nb);
 	clk_unprepare(i2c->pclk);
```

