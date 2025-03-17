---
permalink: /posts/linux_generic_msi
title: "[Linux] GENERIC MSI" 
excerpt: "리눅스 커널의 MSI"
header:
  teaser: /assets/posts/images/note.jpg
categories:
  - Linux
tags:
  - interrupt
  - msi
  - msi-parent
toc: true
---

## 소개

리눅스 커널의 MSI(LPIs) 등록 및 사용 방법에 관한 포스팅(non-pcie) 입니다.<br>
linux kernel 6.12.y 기준, GICv3 이상이며 ITS가 지원되는 환경입니다. 버전에 따라 사용하는 API가 상이할 수 있습니다.<br>
PCIe 전용은 추후에 따로 업로드 예정입니다.<br>

## patch

리눅스 커널에서 dummy로 device를 등록하고 MSI(LPIs)를 매핑하는 패치입니다.<br>
<span style="{{ site.code }}">gic_handle_irq</span> 을 강제로 호출하여 인터럽트 핸들러 호출을 확인했습니다.<br>

### device-tree

임의의 디바이스를 등록합니다.<br>
dummy를 등록할 때 주의해야할 점이 2가지 있습니다.<br>
<br>

첫번째로, dummy device는 memory mapped 된 디바이스로 상정합니다.<br>
즉, 다른말로 memory map이 가능한 영역이어야 합니다.<br>
spec 문서를 참조하여 사용가능한 메모리 영역을 반드시 확인합니다.<br>
두번째로, 할당하는 주소가 다른 영역에 침범(EBUSY)해서는 안됩니다.<br>
kernel의 reserved-memory를 활용하여 이 경우를 배제할 수 있습니다.<br>
<br>

먼저, ITS를 등록합니다.
```
	gic: interrupt-controller@80700000 {
	compatible = "arm,gic-v3";
	#interrupt-cells = <3>;
	#address-cells = <2>;
	#size-cells = <2>;
	ranges;
	interrupt-controller;
	reg = <0x0 0x80700000 0x0 0x10000>,
	      <0x0 0x80780000 0x0 0x100000>;
	interrupts = <GIC_PPI 9 IRQ_TYPE_LEVEL_HIGH 0>;

+	its0: msi-controller@80740000 {
+		compatible = "arm,gic-v3-its";
+		msi-controller;
+		#msi-cells = <1>;
+		reg = <0x0 0x80740000 0x0 0x20000>;
+		dma-nocoherent;
+	};
```
<br>

등록한 <span style="{{ site.code }}">ITS</span> 를 <span style="{{ site.code }}">phandle</span> 로, MSI client는 <span style="{{ site.code }}">msi-parent</span> 라는 속성에서 참조하기 때문입니다.<br>
<span style="{{ site.code }}">msi-cells</span> 을 1로 등록하여, <span style="{{ site.code }}">MSI specifier</span> argument를 추가합니다.
```
+         dummy: dummy@1a00a0000 {
+                 compatible = "steve,dev";
+                 reg = <0x1 0xa00a0000 0x0 0x1000>;
+                 msi-parent = <&its0 0x1>;
+         };
```
<br>

위 주소를 보호하기 위해 <span style="{{ site.code }}">reserved-memory</span> 를 등록해 줍니다.
```
	reserved_memory: reserved-memory {
		#address-cells = <2>;
		#size-cells = <2>;
		ranges;

+		dummy_region {
+			reg = <0x1 0xa00a0000 0x0 0x1000>;
+			no-map;
+			status = "okay";
+		};
	};
```
<br>

이렇게 ITS 및 dummy device 등록이 끝났습니다.<br>

### device driver

device driver는 platform device driver로<br>
dummy device를 probe 합니다. 예제 코드는 다음과 같습니다.
```
// SPDX-License-Identifier: GPL-2.0-or-later
/*
 * Copyright (C) Steve Jeong <steve@how2flow.net>
 */


#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/of.h>
#include <linux/of_platform.h>
#include <linux/io.h>
#include <linux/interrupt.h>
#include <linux/timer.h>
#include <linux/jiffies.h>
#include <linux/slab.h>
#include <linux/msi.h>
#include <linux/irq.h>
#include <linux/irqdomain.h>


#define DRIVER_NAME "dummy_driver"

struct dummy_dev {
	void __iomem *base;
	int msi_irq;
};

static irqreturn_t dummy_msi_irq_handler(int irq, void *dev_id)
{
	pr_info("hello (MSI LPI)\n");

	return IRQ_HANDLED;
}

static void dummy_write_msi_msg(struct msi_desc *desc, struct msi_msg *msg)
{
	/* In a real implementation, msg would be programmed with the ITS doorbell
	 * address and the allocated LPI vector.
	 * For this dummy driver, we simply zero them.
	 */
	msg->address_lo = 0;
	msg->address_hi = 0;
	msg->data = 0;
}

static int dummy_msi_probe(struct platform_device *pdev)
{
	struct dummy_dev *dev;
	struct resource *res;
	int ret;

	dev = devm_kzalloc(&pdev->dev, sizeof(*dev), GFP_KERNEL);
	if (!dev)
		return -ENOMEM;

	res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
	if (!res) {
		dev_err(&pdev->dev, "failed to get memory resource\n");
		return -ENODEV;
	}

	dev->base = devm_ioremap_resource(&pdev->dev, res);
	if (IS_ERR(dev->base))
		return PTR_ERR(dev->base);

	if (!pdev->dev.msi.domain) {
		dev_err(&pdev->dev,
		        "MSI domain is not set; please check the msi-parent property in DT\n");
		return -ENODEV;
	}

	/*
	 * Allocate one MSI IRQ using the upstream API.
	 * The device tree's "msi-parent" property is used by the MSI parent driver
	 * (irq-gic-v3-its-msi-parent.c) to parse the ITS phandle and MSI specifier.
	 */
	ret = platform_device_msi_init_and_alloc_irqs(&pdev->dev, 1, dummy_write_msi_msg);
	if (ret < 0) {
		dev_err(&pdev->dev, "failed to allocate MSI IRQs: %d\n", ret);
		return ret;
	}

	dev->msi_irq=msi_get_virq(&pdev->dev, 0);

	ret = devm_request_irq(&pdev->dev, dev->msi_irq, dummy_msi_irq_handler,
	                       0, DRIVER_NAME, dev);
	if (ret) {
		dev_err(&pdev->dev, "failed to request MSI IRQ %d, ret: %d\n", dev->msi_irq, ret);
		return ret;
	}

	platform_set_drvdata(pdev, dev);
	dev_info(&pdev->dev, "Dummy MSI device probed, allocated MSI IRQ %d\n", dev->msi_irq);

	return 0;
}

static void dummy_msi_remove(struct platform_device *pdev)
{
	dev_info(&pdev->dev, "Dummy MSI device removed\n");
}

static const struct of_device_id dummy_msi_of_match[] = {
	{ .compatible = "steve,dev", },
	{ },
};

MODULE_DEVICE_TABLE(of, dummy_msi_of_match);

static struct platform_driver dummy_msi_driver = {
	.probe  = dummy_msi_probe,
	.remove = dummy_msi_remove,
	.driver = {
		.name           = DRIVER_NAME,
		.of_match_table = dummy_msi_of_match,
	},
};

module_platform_driver(dummy_msi_driver);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Steve Jeong");
MODULE_DESCRIPTION("Dummy device driver using ITS-based MSI (LPIs)");
```
<br>

위 드라이버를 추가하고 Makefile 레시피에 추가합니다.<br>
인터럽트 등록이 성공적으로 된 것을 확인할 수 있습니다.
```
$ cat /proc/interrupts
```
```
           CPU0
 23:       1345     GICv3  30 Level     arch_timer

 ...

 81:          0  ITS-pMSI-1a00a0000.dummy   0 Edge      dummy_driver
 82:          0     GICv3  78 Level     16900000.spi

 ...

Err:          0
```
<br>

추후에 실제로 MSI를 트리거하여 인터럽트 핸들러를 호출하는 포스팅을 게시할 예정입니다.<br>
