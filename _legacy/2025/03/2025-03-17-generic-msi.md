---
permalink: /legacy/linux_generic_msi
title: "[Linux] GENERIC MSI" 
excerpt: "리눅스 커널의 MSI"
header:
  teaser: /assets/images/legacy/note.jpg
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
 * Copyright (C) 2025 Steve Jeong <steve@how2flow.net>
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

### msi trigger

gic의 its와 msi 동작을 이해하기 위해서는 its command에 대한 이해가 선행되어야 합니다.<br>
its command의 경우에는 GICv3_v4 아키텍처 문서에 상세히 기술되어 있습니다.<br>

its에서 msi를 mapping하고 trigger 하기 위해서는 4가지 정보가 필요합니다.
```
1. DeviecID
2. EventID
3. ICID
4. pINTID
```
<br>

<B>DeviceID</B> 란 msi를 보낸 디바이스를 구분하는 ID 값 입니다.<br>
device-tree에서 <span style="{{ site.code }}">msi-parent</span> tuple의 2번째 (index 1) 값이 해당됩니다.<br>
물론, phandle (msi-controller node) 의 <span style="{{ site.code }}">#msi-cells</span> 값은 1이어야 합니다.<br>
<br>

<B>EventID</B> 는 device가 여러 종류의 인터럽트를 요청하려 할때 달라집니다.<br>
인터럽트 하나만 다룬다면 0값이 되겠지만 2가지 이상의 인터럽트 핸들링을 요구 한다면 순차적으로 0, 1, ... 할당되는 구조입니다.<br>
<br>

<B>ICID</B> 는 collection id 인데, 인터럽트를 처리할 target을 결정하는 것입니다.<br>
커널의 <span style= "{{ site.code }}">irq_set_affinity_hint</span> 과 같은 헬퍼 함수를 통해 값을 변경할 수 있습니다.<br>
<br>

<B>pINTID</B> 는 msi와 매핑할 인터럽트 ID 입니다.<br>
실제로 msi를 감지해서, its가 LPI ID로 변환할 때, 이 값으로 변환합니다.<br>
<B>pINTID</B> 역시 커널에서 동적으로 할당합니다.<br>
<br>

커널이 부팅하면서 its 초기화가 끝나면, its command queue에 다음 명령어가 수행된 것을 확인할 수 있습니다.
```
MAPC -> SYNC -> INVALL -> SYNC
```
<br>

각 명령어는 spec 문서에서 자세히 확인이 가능합니다.<br>

#### SW trigger

커널에서는 irq_chip 타입으로 interrupt-controller operations를 관리합니다.<br>
그 중에서 irq_retrigger 라는 function pointer가 있고, 해당 API에 its의 interrupt generate 함수가 plug-in 되어 있습니다.<br>
its의 interrupt generate 동작은 다음과 같습니다.
```
static int its_irq_set_irqchip_state(struct irq_data *d,
                     enum irqchip_irq_state which,
                     bool state)
{
    struct its_device *its_dev = irq_data_get_irq_chip_data(d);
    u32 event = its_get_event_id(d);

    if (which != IRQCHIP_STATE_PENDING)
        return -EINVAL;

    if (irqd_is_forwarded_to_vcpu(d)) {
        if (state)
            its_send_vint(its_dev, event);
        else
            its_send_vclear(its_dev, event);
    } else {
        if (state)
            its_send_int(its_dev, event);
        else
            its_send_clear(its_dev, event);
    }

    return 0;
}

static int its_irq_retrigger(struct irq_data *d)
{
    return !its_irq_set_irqchip_state(d, IRQCHIP_STATE_PENDING, true);
}
```
<br>

드라이버에서 <span style="{{ site.code }}">chip->irq_retrigger(d);</span> 을 호출하면 됩니다.<br>
API가 호출된 시점에서 <span style="{{ site.code }}">hello (MSI LPI)</span> 로그를 확인할 수 있습니다.<br>

#### Trace32

T32 디버거를 연결하여 its command queue를 dump한 후 다음 과정을 통해 인터럽트 트리거를 확인할 수 있습니다.
```
INV(1, 0) -> INT(1, 0)
```
<br>

break를 걸고, 상세한 시퀀스는 다음과 같습니다.
```
&its_cmdq=Data.Long(AD:80740080)&0xFFFFFFFFF000
&its_cwriter=Data.Long(AD:80740088)
&its_creadr=Data.Long(AD:80740090)

// send INV(1,0) //
Data.Set AD:&its_cmdq+its_cwriter %LE %Long 0x000000010000000c
Data.Set AD:<its_cmdq+its_cwriter+0x8 %LE %Long 0x0000000000000000
Data.Set AD:<its_cmdq+its_cwriter+0x10 %LE %Long 0x0000000000000000
Data.Set AD:<its_cmdq+its_cwriter+0x1c %LE %Long 0x0000000000000000

// wait update its_creadr //
&its_cwriter=&its_cwriter+0x20
while(&its_cwriter!=&its_creadr)
	&its_creadr=Data.Long(AD:80740090)

// send INT(1,0) //
Data.Set AD:&its_cmdq+its_cwriter %LE %Long 0x0000000100000003
Data.Set AD:<its_cmdq+its_cwriter+0x8 %LE %Long 0x0000000000000000
Data.Set AD:<its_cmdq+its_cwriter+0x10 %LE %Long 0x0000000000000000
Data.Set AD:<its_cmdq+its_cwriter+0x1c %LE %Long 0x0000000000000000

// wait update its_creadr //
&its_cwriter=&its_cwriter+0x20
while(&its_cwriter!=&its_creadr)
	&its_creadr=Data.Long(AD:80740090)
```
<br>

break를 풀면 <span style="{{ site.code }}">hello (MSI LPI)</span> 로그를 확인할 수 있습니다.<br>

## 정리

이 내용은 GENERIC MSI 를 dummy device를 만들어서 테스트 하는 내용입니다.<br>
실제 하드웨어가 MSI를 지원하는 것이 아니기 때문에, 인터럽트 트리거는 sw generation 방식을 사용할 수밖에 없습니다.<br>
<br>
