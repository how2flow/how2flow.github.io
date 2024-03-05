---
permalink: /posts/fix-gpio-pin/
title: "[Linux] GPIO 고정하기"
excerpt: "device-tree 에서 gpio 모드와 값을 고정하는 방법입니다."
header:
  teaser: /assets/posts/images/fix-gpio-pin.png
categories:
  - Linux
tags:
  - fix
  - gpio
  - devicetree
  - dts
  - device tree
toc: true
---

gpio를 사용해서 무언가를 개발하다보면 특정 핀을 고정하고 싶은 경우가 나타납니다.<br>
기능상의 이유로, 혹은 테스트를 위해 사용해야 합니다.<br>

디바이스 트리를 수정해서 gpio를 고정하는 방법을 소개합니다.<br>
고정하게 되면 sys 노드에서 해당 핀을 export 할 수 없게 됩니다.<br>

[주의] <span style="{{ site.code }}">wiringPi</span> 패키지는 레지스터 주소에 직접 값을 <span style="{{ site.code }}">read/write</span> 하기 때문에 보호할 수 없습니다.<br>

## gpio 고정하기

커널 문서에도 설명이 잘 되어 있습니다.<br>
사실 대부분의 커널 내용은 <span style="{{ site.code }}">Documentation</span> 아래에 서술되어 있습니다.<br>

### 커널 문서 확인하기

gpio 문서를 확인합니다.
```
$ cd linux
$ cat Documentation/devicetree/bindings/gpio/gpio.txt
```
<br>

아래로 쭉 내리다 보면 해당 내용을 볼 수 있습니다.
```
The GPIO chip may contain GPIO hog definitions. GPIO hogging is a mechanism
providing automatic GPIO request and configuration as part of the 
gpio-controller's driver probe function.

Each GPIO hog definition is represented as a child node of the GPIO controller.
Required properties:
- gpio-hog:   A property specifying that this child node represents a GPIO hog.
- gpios:      Store the GPIO information (id, flags, ...) for each GPIO to
              affect. Shall contain an integer multiple of the number of cells
              specified in its parent node (GPIO controller node).
Only one of the following properties scanned in the order shown below.
This means that when multiple properties are present they will be searched
in the order presented below and the first match is taken as the intended
configuration.
- input:      A property specifying to set the GPIO direction as input.
- output-low  A property specifying to set the GPIO direction as output with
              the value low.
- output-high A property specifying to set the GPIO direction as output with
              the value high.

Optional properties:
- line-name:  The GPIO label name. If not present the node name is used.

Example of two SOC GPIO banks defined as gpio-controller nodes:

        qe_pio_a: gpio-controller@1400 {
                compatible = "fsl,qe-pario-bank-a", "fsl,qe-pario-bank";
                reg = <0x1400 0x18>;
                gpio-controller;
                #gpio-cells = <2>;

                line_b {
                        gpio-hog;
                        gpios = <6 0>; 
                        output-low;
                        line-name = "foo-bar-gpio";
                };
        };

        qe_pio_e: gpio-controller@1460 {
                compatible = "fsl,qe-pario-bank-e", "fsl,qe-pario-bank";
                reg = <0x1460 0x18>;
                gpio-controller;
                #gpio-cells = <2>;
        };
```
<br>

내용을 요약하자면, gpio-controller 노드의 child 노드를 만들고 gpio-hog를 정의합니다.<br>
gpios 로 gpio 핀을 지정해주고 모드와 값을 정해줍니다.<br>

### dts 수정하기

만약 내가 gpio3의 5번째 핀을 항상 low 값을 output 하겠다 하면 이렇게 작성합니다.
```
...
#include <dt-bindings/gpio/gpio.h>

...

	gpio3: gpio3@xxxxxxxx {
		compatible = "...";
		gpio-controller;
		...

		always_low {
			gpio-hog;
			gpios = <5 GPIO_ACTIVE_HIGH>;
			output-low;
		};
	};

```
<br>

위 코드에서 <span style="{{ site.code }}">GPIO_ACTIVE_HIGH</span> 를 <span style="{{ site.code }}">GPIO_ACTIVE_LOW</span> 로 바꿔주면 항상 high 값을 output 합니다.<br>
