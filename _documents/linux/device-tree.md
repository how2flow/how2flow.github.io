---
premalink: /documents/linux/device-tree/
title: Devicetree
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
\ {
	compatible = "rockchip,rk3568";
};
```

<span style="{{ site.code }}">cpus</span>: Describing each CPUs.<br>

```
/dts-v1/;

\ {
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

\ {
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

\ {
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

\ {
	compatible = "rockchip,rk3568";

	#address-cells = <2>;
	#size-cells = <2>;

	cpus {
		...
	};

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

\ {
	compatible = "rockchip,rk3568";

	#address-cells = <2>;
	#size-cells = <2>;

	cpus {
		...
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
the <span style="{{ stie.code }}">#address cell</span> of the parent node, the <span style="{{ site.code }}">#address cell</span> of the child node, <br>
and the <span style="{{ site.code }}">#size cell</span> of the child node.<br>
<br>

This is an example that show how to use <span style="{{ site.code }}">ranges</span> in sram node.

```
/dts-v1/;

\ {
	compatible = "rockchip,rk3568";

	#address-cells = <2>;
	#size-cells = <2>;

	cpus {
		...
	};

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

\ {
	compatible = "rockchip,rk3568";

	#address-cells = <2>;
	#size-cells = <2>;

	cpus {
		...
	};

	sram@10f000 {
		...
	};

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
		interrupt-parent = <&gic>;
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
