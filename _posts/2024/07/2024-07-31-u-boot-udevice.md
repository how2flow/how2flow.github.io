---
permalink: /posts/u-boot-udevice
title: "[U-BOOT] 드라이버 ID정보와 dts활용" 
excerpt: "U-BOOT 하나의 드라이버로 여러 칩셋을 커버하기 위한 정보 매핑 및 활용"
header:
  teaser: /assets/posts/images/note.jpg
categories:
  - U-BOOT
tags:
  - device-driver
  - u-boot
toc: true
---

## U-BOOT device driver

작성일 기준 최신 u-boot의 구조는 커널의 구조와 유사합니다.<br>
여러가지 칩을 커버하기 위해서는 드라이버의 발전도<br>
여러 칩을 커버하기 위한 방향으로 발전해야 합니다.<br>

하나의 인터페이스를 다양한 환경(soc)에서 적용할 수 있는<br>
최소한의 드라이버를 개발하기 위해서는<br>
soc를 구분할 수 있는 기법이 중요 합니다.<br>

비단, u-boot 드라이버 만의 얘기가 아닙니다.<br>

드라이버 입장에서는 현재 사용중인 soc 자체는 무의미하고,<br>
특정 정보를 파싱해서 적절한 함수를 call 하면 그만입니다.<br>

### parse device-tree

리눅스 커널은 늘어가는 디바이스를 효율적으로 관리하기 위해<br>
device-tree를 도입했습니다.<br>
U-BOOT도 역시 커널을 따라 갑니다.<br>

디바이스 트리의 property를 parsing한 정보대로만 명령을 실행합니다.<br>
아래 몇가지 예제가 있습니다.<br>
(커널에도 역시 비슷한 역할을 하는 API가 존재합니다.)<br>

U-BOOT에서는 dts parsing API <span style="{{ site.code }}">dev_read_TYPE</span> 이 있습니다.
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
이와 같은 경우, reg에는 0x71004000 freq에는 1000000이 저장됩니다.<br>
<br>

### parse udevice id

device를 구분하려면 매핑 테이블이 필요합니다.<br>
U-BOOT에서는 <span style="{{ site.code }}">udevice_id</span> 타입을 많이 사용합니다.<br>
매핑 테이블에서는 주로 compatible과 data로 device를 구분할 수 있습니다.
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
compatible 속성으로 드라이버를 구분하고,<br>
data 속성으로 동일한 드라이버 내에서 동작을 구분할 수 있습니다.<br>
<br>

data 속성을 활용하기 위해서 <span style="{{ site.code }}">of_to_plat</span> 과 연결된<br>
함수 내에서 <span style="{{ site.code }}">dev_get_driver_data(dev)</span> 을 활용할 수 있습니다.<br>
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
<span style="{{ site.code }}">dev_get_priv</span> 은 udevice 의 private 정보를 넘기는 API 입니다.<br>
+ driver_data를 get 하는 시점은 드라이버 <span style="{{ site.code }}">probe</span> 여도 무방합니다.<br>
<br>
