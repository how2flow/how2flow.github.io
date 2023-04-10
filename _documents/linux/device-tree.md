---
premalink: /documents/linux/device-tree/
title: Devicetree
toc: true
toc_sticky: false
---

## info

As each SOC implemented its own code, the amount of code became too large.
Even if it's compatible.<br>

Recognizing the need to operate without modifying kernel code.<br>
This is how the device tree appeared.<br>

## How to write device-tree?

The path to the device tree is `arch/$ARCH/boot/dts/$vendor/..`

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

`/`: root<br>
`node-A, B`: child node of root.<br>
`child-node-A, B`: child node of 'node-B'<br>
`property`: key-values include empty or some bytes.

### Property

Let's configure the device tree with rk3568 of rockchip.
```
```

#### node

`compatible`: Uniquely identify machines. It usually has a "manufacturer,system" value.<br>

```
\ {
	compatible = "rockchip,rk3568";
};
```

`cpus`: Describing each CPUs.<br>

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

`node`: Unit that defines devices.<br>
It has the form `name@unit-address`.<br>
name is ASCII string. Use generic. This is because there are often multiple identical devices.<br>
If you want to add `3com`, use name `ethernet` instead of `3com`.<br>
unit-address is used to access the device. for example, suppose 3com's address is 0x10000000.<br>
`ethernet@0x10000000`<br>

Nodes can be mapped to labels for easy maintenance. `label`: `name@unit-address`<br>
Like `3com: ethernet@0x10000000`.<br>

#### addressing
Devices that can be addressed encode address information into the device tree using the following properties<br>

`reg`<br>
`#address-cells`<br>
`#size-cells`<br>
<br>
`reg` is tuple.<br>
`reg = < addr1 addr1-size addr2 addr2-size ...>

address and field size are variable.<br>
The `#address-cells` and `#size-cells` of the parent node are used to indicate how many cells are in each field.<br>
One field value has a maximum of 32 bits,<br>
32bit system's `#address-cells` and `#size-cells` are 1.<br>
64bit system's `#address-cells` and `#size-cells` are 2.<br>

Add uart0 node.<br>

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

parent node's `#address-cells` is 2 and `#size-cells`is also 2,<br>
The first tuple is [0x0 0xfdd40000]. the second is [0x0 0x1000].<br>
first is address, second is size.<br>
So, i2c0's address is 0xfdd40000 and size is 0x1000.<br>
i2c0: 0xfdd40000 ~ 0xfdd41000<br>

#### Cpu addressing

Each CPU is assigned a unique single ID and does not have a size.

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

#### Memory-mapped devices

Devices such as i2c and uart are mapped to memory addresses in the device tree.

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

#### Non-Memory-mapped device

Have an address range, but can't access it directly from the CPU.<br>
mapped like cpus.

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

#### Ranges

The root node always describes the CPU's perspective on address space.<br>
The child node of the root already uses the address domain of the CPU, so explicit mapping is not required.<br>
<br>
But, nodes that are not direct children of the root don't use the address domain of the CPU.<br>
So, they need to translate addresses from one domain to another with `ranges`
<br>

ranges = <tuple1 tuple2 ...><br>
The tuple value of ranges is determined by<br>
the `#address cell` of the parent node, the `#address cell` of the child node, <br>
and the `#size cell` of the child node.<br>
<br>

This is an example that show how to use `ranges` in sram node.

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
`ranges = <0 0x0 0x0010f000 0x1000>;`<br>

tuple1 = `0`<br>
'0' is address to map.<br>

tuple2 = `0x0 0x0010f000`<br>
'0x0 0x0010f000' is to be mapped address of parent.<br>

tuple3 = `0x1000`<br>
'0x1000' is the size of address to map<br>

In conclusion,<br>
address <0> of sram (sram@0)is mapped root's <0x0 0x0010f000>. size is <0x100>.<br>
scmi_shmem is mapped at sram@0<br>
The scmi_shmem is mapped from 0x0010f000 to 0x0010f100<br>
via 'ranges' in the 'sram@10f000'.

#### Interrupt

How To define and use interrupts in device-tree.<br>
Interrupt information into the device tree using the following properties<br>
<br>

`interrupts`<br>
`interrupt-controller`<br>
`interrupt-parent`<br>
`#interrupt-cells`<br>
<br>

`interrupts` is tuples like 'reg'.<br>
It has informations about interrupt number, interrupt conditions<br>

`interrupt-controller` has no value.<br>
This property specifies that the node is an interrupt device.<br>

`interrupt-parent` include an node what is interrupt-controller.<br>
The node referenced must have an `interrupt-controller` property.<br>

`#interrupt-cells` defines how many cells to represent an interrupt,<br>
such as '#address-cells' or 'size-cells'.

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
