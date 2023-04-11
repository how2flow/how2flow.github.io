---
premalink: /documents/linux/device-tree/
title: Device tree
excerpt: "how to flow device tree. device, device-tree, overlays ..."
toc: true
---

## info

As each SOC implemented its own code, the amount of code became too large.
Even if it's compatible.<br>

Recognizing the need to operate without modifying kernel code.<br>
This is how the device tree appeared.<br>

## How to write device-tree?

The path to the device tree is <span style="{{ site.code }}">arch/$ARCH/boot/dts/$vendor/..</span>

### Format
If you open any dts file in that path, you can check the following codes.

```
/dts-v1/;

/ {
	node-A {
		property-A;
		property-B="string";
		property-C="string1, string2 ... stringn";
		property-D= [01 02 03 04];
	};

	node-B {
		property-A;
		property-B="hello"
		...

		child-node-A {
			child-property-A;
			child-property-B="hello";
			child-property-C="Steve, Jeong";
			...
		};

		child-node-B {
			...
		};
	};

	...

};

```

<span style="{{ site.code }}">/</span> : root<br>
<span style="{{ site.code }}">node-A, B</span> : child node of root.<br>
<span style="{{ site.code }}">child-node-A, B</span> : child node of 'node-B'<br>
<span style="{{ site.code }}">property</span> : key-values include empty or some bytes.

### Property

Let's configure the device tree with rk3568 of rockchip.
```
```

#### name@unit-address

<span style="{{ site.code }}">compatible</span> : Uniquely identify machines. It usually has a "manufacturer,system" value.<br>

```
/ {
	compatible = "rockchip,rk3568";
};
```

<span style="{{ site.code }}">cpus</span>: Describing each CPUs.<br>

```
/dts-v1/;

/ {
	compatible = "rockchip,rk3568";

	cpus {
		cpu0: cpu@0 {
			compatible = "arm, cortex-a55";
		};

		cpu1: cpu@100 {
			compatible = "arm, cortex-a55";
		};

		cpu2: cpu@200 {
			compatible = "arm, cortex-a55";
		};

		cpu3: cpu@300 {
			compatible = "arm, cortex-a55";
		};
	};
};
```

<span style="{{ site.code }}">node</span> : Unit that defines devices.<br>
It has the form <span style="{{ site.code }}">name@unit-address</span> .<br>
name is ASCII string. Use generic. This is because there are often multiple identical devices.<br>
If you want to add <span style="{{ site.code }}">3com</span>, use name <span style="{{ site.code }}">ethernet</span> instead of <span style="{{ site.code }}">3com</span> .<br>
unit-address is used to access the device. for example, suppose 3com's address is 0x10000000.<br>

`ethernet@0x10000000`<br>

Nodes can be mapped to labels for easy maintenance.<br>

`label : name@unit-address`<br>

Like <span style="{{ site.code }}">3com: ethernet@0x10000000</span> .<br>

#### reg
Devices that can be addressed encode address information into the device tree using the following properties<br>

`reg`<br>
`#address-cells`<br>
`#size-cells`<br>

<span style="{{ site.code }}">reg</span> is tuple.<br>

`reg = < addr1 addr1-size addr2 addr2-size ...>`<br>

address and field size are variable.<br>
The <span style="{{ site.code }}">#address-cells</span> and <span style="{{ site.code }}">#size-cells</span> of the parent node are used to indicate how many cells are in each field.<br>
One field value has a maximum of 32 bits,<br>
32bit system's <span style="{{ site.code }}">#address-cells</span> and <span style="{{ site.code }}">#size-cells</span> are 1.<br>
64bit system's <span style="{{ site.code }}">#address-cells</span> and <span style="{{ site.code }}">#size-cells</span> are 2.<br>

##### Add a device addr.
```
/dts-v1/;

/ {
	compatible = "rockchip,rk3568";

	#address-cells = <2>;
	#size-cells = <2>;

	cpus {
		cpu0: cpu@0 {
			compatible = "arm, cortex-a55";
		};

		cpu1: cpu@100 {
			compatible = "arm, cortex-a55";
		};

		cpu2: cpu@200 {
			compatible = "arm, cortex-a55";
		};

		cpu3: cpu@300 {
			compatible = "arm, cortex-a55";
		};
	};

	i2c0: i2c@fdd40000 {
		compatible = "rockchip,rk3568-i2c";
		reg = <0x0 0xfdd40000 0x0 0x1000>;
	};
};
```

parent node's <span style="{{ site.code }}">#address-cells</span> is 2 and <span style="{{ site.code }}">#size-cells</span> is also 2,<br>
The first tuple is <span style="{{ site.code }}">0x0 0xfdd40000</span>. the second is <span style="{{ site.code }}">0x0 0x1000</span> .<br>
first is address, second is size.<br>
So, i2c0's address is 0xfdd40000 and size is 0x1000.<br>
i2c0: 0xfdd40000 ~ 0xfdd41000<br>

##### Add Cpu addr
```
/dts-v1/;

/ {
	compatible = "rockchip,rk3568";

	#address-cells = <2>;
	#size-cells = <2>;

	cpus {
		#address-cells = <2>;
		#size-cells = <0>; /* cpu does not have size */

		cpu0: cpu@0 {
			compatible = "arm, cortex-a55";
			reg = <0x0 0x0>;
		};

		cpu1: cpu@100 {
			compatible = "arm, cortex-a55";
			reg = <0x0 0x100>;
		};

		cpu2: cpu@200 {
			compatible = "arm, cortex-a55";
			reg = <0x0 0x200>;
		};

		cpu3: cpu@300 {
			compatible = "arm, cortex-a55";
			reg = <0x0 0x300>;
		};
	};

	i2c0: i2c@fdd40000 {
		compatible = "rockchip,rk3568-i2c";
		reg = <0x0 0xfdd40000 0x0 0x1000>;
	};
};
```

##### Add mapped addr
```
/dts-v1/;

/ {
	compatible = "rockchip,rk3568";

	#address-cells = <2>;
	#size-cells = <2>;

	/* cpus, ... */

	i2c0: i2c@fdd40000 {
		compatible = "rockchip,rk3568-i2c";
		reg = <0x0 0xfdd40000 0x0 0x1000>;
	};

	uart0: serial@fdd50000 {
		compatible = "rockchip,rk3568-uart";
		reg = <0x0 0xfdd50000 0x0 0x100>;
	};
};
```

##### Add not mapped addr
```
/dts-v1/;

/ {
	compatible = "rockchip,rk3568";

	#address-cells = <2>;
	#size-cells = <2>;

	/* cpus, ... */

	i2c0: i2c@fdd40000 {
		compatible = "rockchip,rk3568-i2c";
		reg = <0x0 0xfdd40000 0x0 0x1000>;
	};

	uart0: serial@fdd50000 {
		compatible = "rockchip,rk3568-uart";
		reg = <0x0 0xfdd50000 0x0 0x100>;
	};

	spi0: spi@fe610000 {
		compatible = "rockchip,rk3568-spi", "rockchip,rk3066-spi";
		reg = <0x0 0xfe610000 0x0 0x1000>;
		#address-cells = <1>;
		#size-cells = <0>;

		spidev: spidev@0 {
			compatible = "odroid,spi-dev";
			reg = <0>;
			spi-max-frequency = <100000000>;
		};
	};
};
```
spidev is a non-memory mapping device and is accessed via spi0.<br>

#### ranges

The root node always describes the CPU's perspective on address space.<br>
The child node of the root already uses the address domain of the CPU, so explicit mapping is not required.<br>
<br>
But, nodes that are not direct children of the root don't use the address domain of the CPU.<br>
So, they need to translate address from one domain to another with <span style="{{ site.code }}">ranges</span> .
<br>

ranges = <tuple1 tuple2 ...><br>
The tuple value of ranges is determined by<br>
the <span style="{{ site.code }}">#address cell</span> of the parent node, the <span style="{{ site.code }}">#address cell</span> of the child node, <br>
and the <span style="{{ site.code }}">#size cell</span> of the child node.<br>

This is an example that show how to use <span style="{{ site.code }}">ranges</span> in sram node.

```
/dts-v1/;

/ {
	compatible = "rockchip,rk3568";

	#address-cells = <2>;
	#size-cells = <2>;

	/* cpus ... */

	sram@10f000 {
		compatible = "mmio-sram";
		reg = <0x0 0x0010f000 0x0 0x100>;
		#address-cells = <1>;
		#size-cells = <1>;
		ranges = <0 0x0 0x0010f000 0x100>;

		scmi_shmem: sram@0 {
			compatible = "arm, scmi-shmem";
			reg = <0x0 0x100>;
		};
	};

	i2c0: i2c@fdd40000 {
		compatible = "rockchip,rk3568-i2c";
		reg = <0x0 0xfdd40000 0x0 0x1000>;
	};

	uart0: serial@fdd50000 {
		compatible = "rockchip,rk3568-uart";
		reg = <0x0 0xfdd50000 0x0 0x100>;
	};

	spi0: spi@fe610000 {
		compatible = "rockchip,rk3568-spi", "rockchip,rk3066-spi";
		reg = <0x0 0xfe610000 0x0 0x1000>;
		#address-cells = <1>;
		#size-cells = <0>;

		spidev: spidev@0 {
			compatible = "odroid,spi-dev";
			reg = <0>;
			spi-max-frequency = <100000000>;
		};
	};
};
```

ranges have tuples<br>
'address to map (child's #address-cells)' 1<br>
'to be mapped address of parent (parent's #address-cells)' 2<br>
'size of address to map (child's #size-cells)' 1<br>
parent is root, child is scmi_shmem.<br>

In the above,<br>
<span style="{{ site.code }}">ranges = <0 0x0 0x0010f000 0x1000>;</span><br>

tuple1 = <span style="{{ site.code }}">0</span><br>
'0' is address to map.<br>

tuple2 = <span style="{{ site.code }}">0x0 0x0010f000</span><br>
'0x0 0x0010f000' is to be mapped address of parent.<br>

tuple3 = <span style="{{ site.code }}">0x1000</span><br>
'0x1000' is the size of address to map<br>

In conclusion,<br>
address <0> of sram (sram@0)is mapped root's <0x0 0x0010f000>. size is <0x100>.<br>
scmi_shmem is mapped at sram@0<br>
The scmi_shmem is mapped from 0x0010f000 to 0x0010f100<br>
via 'ranges' in the 'sram@10f000'.

#### interrupts

How To define and use interrupts in device-tree.<br>
Interrupt information into the device tree using the following properties<br>

`interrupts`<br>
`interrupt-controller`<br>
`interrupt-parent`<br>
`#interrupt-cells`<br>

<span style="{{ site.code }}">interrupts</span> is tuples like 'reg'.<br>
It has informations about interrupt number, interrupt conditions<br>

<span style="{{ site.code }}">interrupt-controller</span> has no value.<br>
This property specifies that the node is an interrupt device.<br>

<span style="{{ site.code }}">interrupt-parent</span> include an node what is interrupt-controller.<br>
The node referenced must have an <span style="{{ site.code }}">interrupt-controller</span> property.<br>

<span style="{{ site.code }}">#interrupt-cells</span> defines how many cells to represent an interrupt,<br>
such as <span style="{{ site.code }}">#address-cells</span> or <span style="{{ site.code }}">size-cells</span> .

```
/dts-v1/;

#include <dt-bindings/interrupt-controller/arm-gic.h>
#include <dt-bindings/interrupt-controller/irq.h>

/ {
	compatible = "rockchip,rk3568";

	#address-cells = <2>;
	#size-cells = <2>;

	/* cpus, sram ...*/

	gic: interrupt-controller@fd400000 {
		compatible = "arm,gic-v3";
		reg = <0x0 0xfd400000 0 0x10000>, /* GICD */
			  <0x0 0xfd460000 0 0x80000>; /* GICR */
		interrupts = <GIC_PPI 9 IRQ_TYPE_LEVEL_HIGH>;
		interrupt-controller;
		#interrupt-cells = <3>;
	};

	i2c0: i2c@fdd40000 {
		compatible = "rockchip,rk3568-i2c";
		reg = <0x0 0xfdd40000 0x0 0x1000>;
		interrupt-parent = <&gic>;
		interrupts = <GIC_SPI 46 IRQ_TYPE_LEVEL_HIGH>;
	};

	uart0: serial@fdd50000 {
		compatible = "rockchip,rk3568-uart";
		reg = <0x0 0xfdd50000 0x0 0x100>;
		interrupt-parent = <&gic>;
		interrupts = <GIC_SPI 116 IRQ_TYPE_LEVEL_HIGH>;
	};

	spi0: spi@fe610000 {
		compatible = "rockchip,rk3568-spi", "rockchip,rk3066-spi";
		reg = <0x0 0xfe610000 0x0 0x1000>;
		interrupts = <GIC_SPI 103 IRQ_TYPE_LEVEL_HIGH>;
		#address-cells = <1>;
		#size-cells = <0>;

		spidev: spidev@0 {
			compatible = "odroid,spi-dev";
			reg = <0>;
			spi-max-frequency = <100000000>;
		};
	};
};
```

<span style="{{ site.code }}">gic</span> is interrupt controller.<br>
<span style="{{ site.code }}">i2c0</span>, <span style="{{ site.code }}">uart0</span>, <span style="{{ site.code }}">spi0</span> have interrupt pin.<br>
interrupt pins are defined by <span style="{{ site.code }}">interrupt</span> and the gic device properties are inherited by the <span style="{{ site.code }}">interrupt-parent</span> .<br>
<span style="{{ site.code }}">spi0</span> doesn't have <span style="{{ site.code }}">interrupt-parent</span> .<br>
In this case, it can inherit interrupt properties from the parent( <span style="{{ site.code }}">/</span> ) node.

#### clocks

<span style="{{ site.code }}">clock</span> is the property that defines the clock of the device.<br>

`clocks`<br>
`clock-name`<br>
`#clock-cells`<br>
`clock-frequency`<br>

<span style="{{ site.code }}">clocks</span> get properties of clock.<br>
<span style="{{ site.code }}">clock-name</span> set clock name.<br>
<span style="{{ site.code }}">#clock-cells</span> how many use cells for setting clock cell.<br>
<span style="{{ site.code }}">clock-frequency</span> set clock frequency.<br>

If you look closely at the names of the properties, you'll see,<br>
The same pattern is repeated over and over again and again.<br>

There are two cases of defining clocks.<br>
If a device has vendor's clock driver or not.<br>
case 1. If yes, compatible vendor's driver<br>
case 2. If no, fixed-clock.<br>

##### compatible vendor's driver

define clocks using a clock device driver of rk3568.

```
/dts-v1/;

#include <dt-bindings/clock/rk3568-cru.h>
#include <dt-bindings/interrupt-controller/arm-gic.h>
#include <dt-bindings/interrupt-controller/irq.h>

/ {
	compatible = "rockchip,rk3568";

	#address-cells = <2>;
	#size-cells = <2>;

	/* cpus, sram, gic ...*/

	pmucru: clock-controller@fdd00000 {
		compatible = "rockchip, rk3568-pmucru";
		reg = <0x0 0xfdd00000 0x0 0x1000>;
		rockchip,grf = <&grf>;
		rockchip,pmugrf = <&pmugrf>;
		#clock-cells = <1>;
		#reset-cells <1>;

		assigned-clocks = <&pmucru SCLK_32K_IOE>;
		assigned-clock-parents = <&pmucru CLK_RTC_32K>;
	};

	cru: clock-controller@fdd20000 {
		compatible = "rockchip, rk3568-cru";
		reg = <0x0 0xfdd20000 0x0 0x1000>;
		rockchip,grf = <&grf>;
		#clock-cells = <1>;
		#reset-cells <1>;

		assigned-clocks =
			...

		assigned-clock-rates =
			...

		assigned-clock-parents =
			...
	};

	i2c0: i2c@fdd40000 {
		compatible = "rockchip,rk3568-i2c";
		reg = <0x0 0xfdd40000 0x0 0x1000>;
		clocks = <&pmucru CLK_I2C0>, <&pmucru PCLK_I2C0>;
		clock-names = "i2c", "pclk";
		interrupt-parent = <&gic>;
		interrupts = <GIC_SPI 46 IRQ_TYPE_LEVEL_HIGH>;
	};

	uart0: serial@fdd50000 {
		compatible = "rockchip,rk3568-uart";
		reg = <0x0 0xfdd50000 0x0 0x100>;
		clocks = <&pmucru SCLK_UART0>, <&pmucru PCLK_UART0>;
		clock-names = "baudclk", "apb_pclk";
		interrupt-parent = <&gic>;
		interrupts = <GIC_SPI 116 IRQ_TYPE_LEVEL_HIGH>;
	};

	spi0: spi@fe610000 {
		compatible = "rockchip,rk3568-spi", "rockchip,rk3066-spi";
		reg = <0x0 0xfe610000 0x0 0x1000>;
		interrupts = <GIC_SPI 103 IRQ_TYPE_LEVEL_HIGH>;
		#address-cells = <1>;
		#size-cells = <0>;
		clocks = <&cru CLK_SPI0>, <&cru PCLK_SPI0>;
		clock-names = "spiclk", "apb_pclk";

		spidev: spidev@0 {
			compatible = "odroid,spi-dev";
			reg = <0>;
			spi-max-frequency = <100000000>;
		};
	};
};
```

##### fixed-clock

define clocks using fixed-clock. This is not vendor related.

```
/dts-v1/;

#include <dt-bindings/clock/rk3568-cru.h>
#include <dt-bindings/interrupt-controller/arm-gic.h>
#include <dt-bindings/interrupt-controller/irq.h>

/ {
	compatible = "rockchip,rk3568";

	#address-cells = <2>;
	#size-cells = <2>;

	/* cpus, sram, gic, pmucru, cru, i2c0, uart0 ... */

	mcp2515_clk: mcp2515_clk {
		compatible = "fixed-clock";
		#clock-cells = <0>;
		clock-frequency = <8000000>;
	};

	spi0: spi@fe610000 {
		compatible = "rockchip,rk3568-spi", "rockchip,rk3066-spi";
		reg = <0x0 0xfe610000 0x0 0x1000>;
		interrupts = <GIC_SPI 103 IRQ_TYPE_LEVEL_HIGH>;
		#address-cells = <1>;
		#size-cells = <0>;
		clocks = <&cru CLK_SPI0>, <&cru PCLK_SPI0>;
		clock-names = "spiclk", "apb_pclk";

		spidev: spidev@0 {
			compatible = "odroid,spi-dev";
			reg = <0>;
			spi-max-frequency = <100000000>;
		};

		mcp2515: mcp2515@0 {
			compatible = "microchip, mcp2515";
			clocks = <&mcp2515_clk>;
			reg = <1>;
			spi-max-frequency = <10000000>;
		};
	};
};
```

Add the mcp2515 node and mcp2515 clock.<br>
However, it is better to use overlays than to add them directly to the tree.

#### pinctrl

It is responsible for assigning devices to pins.<br>

`pinctrl-names`<br>
`pinctrl-n`<br>

<span style="{{ site.code }}">pinctrl-names</span> and <span style="{{ site.code }}">pinctrl-n</span> can be defined as one or more strings.<br>
<span style="{{ site.code }}">pinctrl-names</span> defines the state of pin.<br>
basically, state are 'default', 'init', 'idle', 'sleep'<br>
in <span style="{{ site.code }}">include/linux/pinctrl/pinctrl-state.h</span> .<br>

You can modify and customize pin states by modifying kernel code.<br>
Related information will be posted in another post.<br>
<span style="{{ site.code }}">pinctrl-n</span> Defines the pin information.<br>
If the pin is in the n'th <span style="{{ site.code }}">pinctrl-names state</span> ,<br>
the device is assigned a pin of <span style="{{ site.code }}">pintrl-n</span> .

```
device: device@10000000 {
	...
	pinctrl-names = "default", "sleep";
	pinctrl-0 = <&pin1>; /* if pin state is default, */
	pinctrl-1 = <&pin2>; /* if pin state is sleep, */
	...
};
```

Pinctrl is very important for I/O setting.<br>
This is an example of rk3568.

```
/dts-v1/;

#include <dt-bindings/clock/rk3568-cru.h>
#include <dt-bindings/interrupt-controller/arm-gic.h>
#include <dt-bindings/interrupt-controller/irq.h>
#include <dt-bindings/pinctrl/rockchip.h>

/ {
	compatible = "rockchip,rk3568";

	#address-cells = <2>;
	#size-cells = <2>;

	/* cpus, sram, gic, pmucru, cru ... */

	i2c0: i2c@fdd40000 {
		compatible = "rockchip,rk3568-i2c";
		reg = <0x0 0xfdd40000 0x0 0x1000>;
		clocks = <&pmucru CLK_I2C0>, <&pmucru PCLK_I2C0>;
		clock-names = "i2c", "pclk";
		interrupt-parent = <&gic>;
		interrupts = <GIC_SPI 46 IRQ_TYPE_LEVEL_HIGH>;
		pinctrl-names = "default";
		pinctrl-0 = <&i2c0_xfer>;
	};

	uart0: serial@fdd50000 {
		compatible = "rockchip,rk3568-uart";
		reg = <0x0 0xfdd50000 0x0 0x100>;
		clocks = <&pmucru SCLK_UART0>, <&pmucru PCLK_UART0>;
		clock-names = "baudclk", "apb_pclk";
		interrupt-parent = <&gic>;
		interrupts = <GIC_SPI 116 IRQ_TYPE_LEVEL_HIGH>;
		pinctrl-names = "default";
		pinctrl-0 = <&uart0_xfer>;
	};

	spi0: spi@fe610000 {
		compatible = "rockchip,rk3568-spi", "rockchip,rk3066-spi";
		reg = <0x0 0xfe610000 0x0 0x1000>;
		interrupts = <GIC_SPI 103 IRQ_TYPE_LEVEL_HIGH>;
		#address-cells = <1>;
		#size-cells = <0>;
		clocks = <&cru CLK_SPI0>, <&cru PCLK_SPI0>;
		clock-names = "spiclk", "apb_pclk";
		pinctrl-names = "default", "high_speed";
		pinctrl-0 = <&spi0m0_cs0 &spi0m0_cs1 &spi0m0_pins>;
		pinctrl-1 = <&spi0m0_cs0 &spi0m0_cs1 &spi0m0_pins_hs>;

		spidev: spidev@0 {
			compatible = "odroid,spi-dev";
			reg = <0>;
			spi-max-frequency = <100000000>;
		};
	};
};
```

#### aliases

Nodes typically refer to the entire path.<br>
To refer to uart0, need path.<br>
path is <span style="{{ site.code }}">/serial@fdd50000</span> .<br>

A longer path will be required to reference the child nodes of a particular node.<br>
So it gives specific nodes an alias through <span sytle="{{ site.code }}">aliases</span> .<br>

```
/ {
	...

	aliases {
		serial0 = &uart0;
	}
	...

};
```
Node switching is also possible when utilized well.
```
/ {
	...

	aliases {
		serial0 = &uart1;
	}
	...

};
```

## Example

Let me give you a complicated example.<br>

### PCI

PCi has a unique memory mapping method and various properties.
This is an pcie2x1 example of rk3568

```
	pcie2x1: pcie@fe260000 {
		compatible = "rockchip,rk3568-pcie", "snps,dw-pcie";
		#address-cells = <3>;
		#size-cells = <2>;
		bus-range = <0x0 0xf>;
		clocks = <&cru ACLK_PCIE20_MST>, <&cru ACLK_PCIE20_SLV>,
			<&cru ACLK_PCIE20_DBI>, <&cru PCLK_PCIE20>,
			<&cru CLK_PCIE20_AUX_NDFT>;
		clock-names = "aclk_mst", "aclk_slv",
			  "aclk_dbi", "pclk", "aux";
		device_type = "pci";
		interrupts = <GIC_SPI 75 IRQ_TYPE_LEVEL_HIGH>,
			<GIC_SPI 74 IRQ_TYPE_LEVEL_HIGH>,
			<GIC_SPI 73 IRQ_TYPE_LEVEL_HIGH>,
			<GIC_SPI 72 IRQ_TYPE_LEVEL_HIGH>,
			<GIC_SPI 71 IRQ_TYPE_LEVEL_HIGH>;
		interrupt-names = "sys", "pmc", "msg", "legacy", "err";
		#interrupt-cells = <1>;
		interrupt-map-mask = <0 0 0 7>;
		interrupt-map = <0 0 0 1 &pcie2x1_intc 0>,
			<0 0 0 2 &pcie2x1_intc 1>,
			<0 0 0 3 &pcie2x1_intc 2>,
			<0 0 0 4 &pcie2x1_intc 3>;
		linux,pci-domain = <0>;
		num-ib-windows = <6>;
		num-ob-windows = <2>;
		max-link-speed = <2>;
		msi-map = <0x0 &its 0x0 0x1000>;
		num-lanes = <1>;
		phys = <&combphy2_psq PHY_TYPE_PCIE>;
		phy-names = "pcie-phy";
		power-domains = <&power RK3568_PD_PIPE>;
		ranges = <0x00000800 0x0 0x00000000 0x3 0x00000000 0x0 0x800000
			0x81000000 0x0 0x00800000 0x3 0x00800000 0x0 0x100000
			0x83000000 0x0 0x00900000 0x3 0x00900000 0x0 0x3f700000>;
		reg = <0x3 0xc0000000 0x0 0x400000>,
			<0x0 0xfe260000 0x0 0x10000>;
		reg-names = "pcie-dbi", "pcie-apb";
		resets = <&cru SRST_PCIE20_POWERUP>;
		reset-names = "pipe";
		status = "disabled";

		pcie2x1_intc: legacy-interrupt-controller {
			interrupt-controller;
			#address-cells = <0>;
			#interrupt-cells = <1>;
			interrupt-parent = <&gic>;
			interrupts = <GIC_SPI 72 IRQ_TYPE_EDGE_RISING>;
		};
	};
```

Except for almost the attributes you've never seen before,<br>
There's something in this code that you need to look at as unique and special.<br>

<span style ="{{ site.code }}">bus-ranges</span> and <span style="{{ site.code }}">#address-cells = 3</span> .<br>

<span style ="{{ site.code }}">bus-ranges</span> is bus numbering of pci.<br>
```
Since the PCI bus has a structure that puts a bridge circuit between the CPU and the bus,
VESA Unlike the local bus, even if the type of CPU is different,
it can be connected to any CPU as long as the corresponding bridge circuit is equipped.
[네이버 지식백과] PCI [peripheral component interconnect] (IT용어사전, 한국정보통신기술협회)
```
first cell is index of bus and second cell provides the maximum bus number for all sub-pci buses.<br>

<span style ="{{ site.code }}">#address-cells = 3</span> is for translation address.<br>
Similar to the local bus described earlier,<br>
the PCI address space is completely separate from the CPU address space.<br>
So It must be imported from a PCI address to a CPU address.<br>

```
/ {
	#address-cells = <2>;
	...

	pcie2x1 {
		...

		#address-cells = <3>;
		#size-cells = <2>;
		...

		ranges = <0x00000800 0x0 0x00000000 0x3 0x00000000 0x0 0x800000 /* first */
			0x81000000 0x0 0x00800000 0x3 0x00800000 0x0 0x100000 /* second */
			0x83000000 0x0 0x00900000 0x3 0x00900000 0x0 0x3f700000>; /* third */
	};
	...

};
```

Why does the device-tree need three 32-bit cells to address PCI?<br>
pci has 3cell. <span style="{{ site.code }}">phys.hi</span> , <span style="{{ site.code }}">phys.mid</span> and <span style="{{ site.code }}">phys.low</span> .<br>
Of course, pci address-bit is 64-bit, what is encoded in the address is <span style="{{ site.code }}">phys.mid</span> and <span style="{{ site.code }}">phys.low</span> .<br>
<span style="{{ site.code }}">phys.hi</span> is feild of pci properties.

```
# pci feild (phys.hi)

npt000ss bbbbbbbb dddddfff rrrrrrrr

n: Relocateable Flag. (0 means relocation, 1 means impossible)
p: Prefetch Flag. (0 means impossible 1 means possible)
t: Aliased Flag.
ss: space code
  00: compositional space
  01: I/O space
  10: 32bit memory space
  11: 64bit memory space
bbbbbbbb: sub-bus
ddddd: IDSEL number.
fff: function number.
rrrrrrrr: registration number. used in configuration cycle.
```

important flag is 'p' and 'ss'.<br>
<span style="{{ site.code }}">p</span> and <span style="{{ site.code }}">ss</span> determine the PCI address space to access.<br>

Look at the first of ranges,

```
# first

0x00000800 0x0 0x00000000 0x3 0x00000000 0x0 0x800000
```
pci field: <span style="{{ site.code }}">0x00000800</span><br>
```
npt000ss bbbbbbbb dddddfff rrrrrrrr
00000000 00000000 00001000 00000000
```

PCI address: <span style="{{ site.code}}">0x0 0x00000000</span><br>
It is <span style="{{ site.code }}">PCI@0x000000000</span> .<br>

CPU address to map: <span style="{{ site.code}}">0x3 0x00000000</span><br>
It is <span style="{{ site.code }}">CPU@0x300000000</span> .<br>

PCI size: <span style="{{ site.code}}">0x0 0x800000</span><br>
0x800000 is 8MB.<br>

Map PCI address 0x000000000 to CPU address 0x300000000.<br>
The size is 8 MB.<br>
<span style="{{ site.code }}">PCI@{0x000000000 ~ 0x000800000} => CPU@{0x300000000 ~ 0x300800000}</span> .<br>

The second and third interpretations are the same.<br>
