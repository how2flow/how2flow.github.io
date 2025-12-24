---
permalink: /legacy/rpmsg
title: "RPMsg 포팅" 
excerpt: "rpmsg 포팅 노트"
header:
  teaser: /assets/images/legacy/note.jpg
categories:
  - Linux
tags:
  - openamp
  - rpmsg
  - zephyr
toc: true
---

## 소개

멀티코어 환경에서 가볍고 효율적인 프로세서 간 통신(IPC)을 구현하기 위해 사용되는<br>
표준 프로토콜이 바로 RPMsg(Remote Processor Messaging) 입니다.<br>
주로 Linux + MCU(또는 RTOS) 구조에서 널리 활용되며, 임베디드 시스템의 이종 프로세서 간 메시지 교환을<br>
단순하고 안정적으로 만들어 줍니다.<br>

### RPMsg 기본 구조

RPMsg는 OpenAMP 프레임워크 위에서 동작합니다.<br>
구성요소는 다음과 같습니다.<br>

#### virtio

가상의 장치를 추상화 하는 인터페이스 입니다.<br>

#### Vring

실제 데이터 버퍼의 링 구조 큐<br>
송/수신 메세지를 큐 기반으로 관리 합니다.<br>

#### Remoteproc

Remote device를 로드/부팅/관리 하는 기능<br>
크게 start, stop, kick 3가지 기능이 있으며, start/stop은 Remote device 제어,<br>
kick은 통신을 하기 위한 doorbell(notify)역할을 한다.<br>
각 벤더마다의 시나리오를 구축할 수 있도록 플랫폼 드라이버가 필요하다.<br>
Master 쪽에서 핵심적으로 구현이 필요한 부분입니다.<br>

#### RPMsg endpoint

실제 메세지를 주고 받는 채널 입니다.<br>
Linux와 RTOS 각각에서 동일한 이름을 가진 endpoint 끼리 연결됩니다.<br>

### RPMsg 포팅 이유

멀티코어 간 IPC 직접 구현할 필요 없이 표준 API를 제공합니다.<br>
공유 메모리 기반 구조로 빠른 전송이 가능합니다.<br>
Linux/Zephyr등 주요 OS를 지원합니다.<br>

## Porting Note

포팅 시나리오는 두 클러스터 간 rpmsg를 활용한 통신입니다.<br>
RPMsg에서는 Master와 Remote 장치로 나누어 지는데 보통 Master device에는 Linux,<br>
Remote device는 Zephyr같은 RTOS가 돌아가는 환경이 주가 됩니다.<br>
<br>

먼저, master를 위한 driver를 개발하고, remote는 zephyr 환경에서 운용하도록 하겠습니다.
```
chip: tcc807x
linux: 5.10 (Sub-Core, RPMsg master)
zephyr: 3.7 (Main Core, RPMsg remote)
```
<br>

1차 시나리오는 remoteproc 드라이버에서 start/stop을 제외하고,<br>
kick만 구현하여 doorbell interrupt만 지원합니다.<br>
우선, 제대로 동작이 되는지 확인하는 단계가 되겠습니다.<br>
따라서, Remote device의 <B>self-boot</B> 상황을 가정하고, carveouts를 static 하게 제공합니다.<br>
virtio 기반으로 shared memory를 쓰기 때문에, <B>dcache coherency</B>를 반드시 보장해야 합니다.<br>
(cache invalidate / cache flush)<br>
<br>

시나리오상, <span style="{{ site.code }}">resource_table</span> 은 remote 최종 형상으로부터 결정되기 때문에,<br>
remote부터 작업합니다.<br>

### Remote: zephyr examples

zephyr 같은 경우에는 기본적으로 openamp library가 제공되어 있고,<br>
sample이 완성되어 있습니다.<br>
openamp_rsc_table 예제로, linux kernel과 통신예제를 확인할 수 있습니다.<br>
<br>

#### cache coherency

그 이전에, zephyr에서도 cache coherency를 보장해줘야 합니다.<br>
다음 2가지 config가 활성화 되어 있어야 합니다.
```
CONFIG_CACHE_MANAGEMENT=y
CONFIG_OPENAMP_WITH_DCACHE=y
```
<br>

반영 여부는 west build 후, <span style="{{ site.code }}">build/zephyr/.config</span> 에서 확인할 수 있습니다.<br>
위 config들이 반영되어 있어야 zephyr는 <span style="{{ site.code }}">cache invalidate / flush</span> 를 제대로 수행합니다.<br>
<br>

#### self-boot 시나리오

기본적으로, auto-boot기준으로 드라이버가 개발되어 있기 때문에 self-boot에 맞게<br>
일부 수정해 주어야 합니다.<br>
특히 library도 수정해주어야 하기 때문에 저같은 경우에는 sample을 따로 분리해서 개발했습니다.<br>
<span style="{{ site.code }}">lib/open-amp/resource_table.h</span> 파일에 다음 부분을 수정해 주어야 합니다.<br>
(참고) 저같은 경우, lib 아래 파일을 수정하는 것이 껄끄러워서, sample을 따로 분리해서 작업했습니다.<br>
```
 boards/telechips/tcc807x/tcc8070_lpd4x322.dts                    |   7 +
 dts/arm64/telechips/tcc807x-rproc.dtsi                           |  12 +
 samples/subsys/ipc/tcc_rsc_table/CMakeLists.txt                  |  13 +
 samples/subsys/ipc/tcc_rsc_table/README.rst                      | 127 ++++++++++
 samples/subsys/ipc/tcc_rsc_table/boards/tcc8070_lpd4x322.conf    |   2 +
 samples/subsys/ipc/tcc_rsc_table/boards/tcc8070_lpd4x322.overlay |  25 ++
 samples/subsys/ipc/tcc_rsc_table/prj.conf                        |  13 +
 samples/subsys/ipc/tcc_rsc_table/sample.yaml                     |   6 +
 samples/subsys/ipc/tcc_rsc_table/src/main_remote.c               | 405 +++++++++++++++++++++++++++++++
 samples/subsys/ipc/tcc_rsc_table/src/resource_table.h            |  84 +++++++
 soc/telechips/tcc807x/mmu_regions.c                              |  11 +-
```
<br>

일단 수정 내용을 살펴보면,
```
...
#define VRING_RX_ADDRESS        -1  /* allocated by Master processor */
#define VRING_TX_ADDRESS        -1  /* allocated by Master processor */
#define VRING_BUFF_ADDRESS      -1  /* allocated by Master processor */
```
<br>

이 부분을 static한 주소할당이 유용하도록 제공해야 합니다.
```
#define VRING_RX_ADDRESS        0x200c9000UL  /* reserved address (self-boot) */
#define VRING_TX_ADDRESS        0x200c8000UL  /* reserved address (self-boot) */
#define VRING_BUFF_ADDRESS      0x200ca000UL  /* reserved address (self-boot) */
```
<br>

#### section: resource_table

zephyr 빌드 후, resource_table 영역은 다음 명령어로 확인할 수 있습니다.
```
$ nm build/zephyr/zephyr_tcc_rsc_table.elf | grep resource_table
00000000200c63b8 d resource_table
```
<br>

carveout 주소들이 어떻게 정해졌냐면,<br>
메모리 영역을 잡는데 가장 중요한 것은 resource_table 이며, 사실 carveout들은 이처럼<br>
resource table과 가까이 붙어 있을 필요는 없습니다.<br>
처음에는 임의의 주소를 정해 놓고 나서 빌드 후, 0x200c63b8이라는 주소를 얻게 되었고,<br>
나중에 carveouts 주소를 0x200c8000--0x200ce0000 만큼 할당한 것입니다.<br>
<br>

#### zephyr mmu region

zephyr는 mmu region을 추가해 주어야 합니다.<br>
각 벤더에 정의된 mmu_region을 추가합니다.
```
...

arm_mmu_region mmu_regions[] = {
	...

	MMU_REGION_FLAT_ENTRY("VRING0", 0x200c8000, 0x1000,
	MT_NORMAL | MT_P_RW_U_NA | MT_DEFAULT_SECURE_STATE),

	MMU_REGION_FLAT_ENTRY("VRING1", 0x200c9000, 0x1000,
	MT_NORMAL | MT_P_RW_U_NA | MT_DEFAULT_SECURE_STATE),

	MMU_REGION_FLAT_ENTRY("VDEV0BUFFER", 0x200ca000, 0x4000,
	MT_NORMAL | MT_P_RW_U_NA | MT_DEFAULT_SECURE_STATE),

	...
};
...
```
<br>

zephyr가 모르는 영역에 접근하게 되면 panic이 발생하기 때문입니다.<br>
device tree는 다음과 같이 구성하였습니다.<br>
<br>

dtsi
```
/*
 * Copyright (C) 2024 Telechips Inc.
 *
 * SPDX-License-Identifier: Apache-2.0
 */

&{/} {
	chosen {
		zephyr,ipc = &a55mp_a55sp_mbox;
		zephyr,ipc_shm = &vdev0buffer;
	};
};
```
<br>

dts
```
...
	reserved_memory: reserved-memory {
		#address-cells = <2>;
		#size-cells = <2>;
		ranges;
	};
...
```
<br>

overlay
```
/*
 * Copyright (C) 2025 Telechips Inc.
 *
 * SPDX-License-Identifier: Apache-2.0
 */

&reserved_memory {
	vdev0vring0: vdev0vring0@200c8000 {
		compatible = "zephyr,memory-region", "shared-dma-pool";
		reg = <0x0 0x200c8000 0x0 0x1000>;
		zephyr,memory-region = "VRING0";
	};

	vdev0vring1: vdev0vring1@200c9000 {
		compatible = "zephyr,memory-region", "shared-dma-pool";
		reg = <0x0 0x200c9000 0x0 0x1000>;
		zephyr,memory-region = "VRING1";
	};

	vdev0buffer: vdev0buffer@200ca000 {
		compatible = "zephyr,memory-region", "shared-dma-pool";
		reg = <0x0 0x200ca000 0x0 0x4000>;
		zephyr,memory-region = "VDEV0BUFFER";
	};
};
```
<br>


### Master: Linux driver

리눅스에는 기본적으로 rpmsg 프레임워크를 제공합니다.
```
$ ls drivers/rpmsg
Makefile mtk_rpmsg.c rpmsg_char.c rpmsg_core.c
Kconfig virtio_rpmsg_bus.c rpmsg_internal.h 
```
<br>

리눅스에는 기본적으로 remoteproc 프레임워크도 제공합니다.
```
$ ls drivers/remoteproc
Kconfig mtk_common.h mtk_scp.c mtk_scp_ipi.c
Makefile omap_remoteproc.c omap_remoteproc.h
remoteproc_cdev.c remoteproc_core.c remoteproc_coredump.c
remoteproc_debugfs.c remoteproc_elf_helpers.h remoteproc_elf_loader.c
remoteproc_internal.h remoteproc_sysfs.c remoteproc_virtio.c
```
<br>

기본적으로 Remote device <B>auto-boot</B> 시나리오 기준으로 맞추어진 프레임 워크입니다.<br>
tcc807x 칩을 기준으로, remote device는 master device가 부팅하기 이전에 부팅이 되어 있을 경우가<br>
대부분이고, remote device의 생명주기를 관리할 예정이 없기 때문에, <B>self-boot</B> 기준으로<br>
remoteproc platform driver를 개발합니다.<br>

#### self-boot 시나리오

RPMsg 전체 시나리오는 다음과 같습니다.<br>

```
1. 커널 부팅 시, Device Tree에 명시된 "telechips,tcc807x-rproc" compatible 문자열과 일치하는<br>
   tcc_rproc_driver의 probe 함수가 호출됩니다.

2. rproc_alloc(): 원격 프로세서(remote processor)를 표현하는 struct rproc 객체를 생성합니다.
   이 객체는 원격 프로세서의 라이프사이클을 관리하는 핸들입니다. self-boot 이므로, kick 만 구현합니다.
   Device Tree의 memory-region 속성을 읽어 리소스 테이블(rsc-table)을 포함한 공유 메모리 영역을 memremap으로 매핑합니다.
   이 단계에서 priv->rsc_table에 리소스 테이블의 가상 주소가 저장됩니다.
   그리고 rproc_add_carveout()를 호출하여 사전에 carveouts가 있음을 virtio_rpmsg 프레임 워크에 알려야 합니다.

3. rproc_boot() 함수는 self-boot 구현이므로 호출 되기 전, rproc->state가 RPROC_DETACHED 상태여야 하고,
   auto-boot 옵션을 꺼서 펌웨어를 로드하는 대신 rproc_actuate() 함수를 호출합니다.

4. 리소스 테이블에서 type이 RSC_VDEV인 항목을 발견하면, rproc_handle_vdev() 함수가 호출됩니다.
   이 항목이 바로 virtio 장치를 정의합니다.
   (추가) 만약, 2번 과정에서 rproc_add_carveout()을 호출하지 않았다면,
   이 과정에서 커널은 dma_alloc_coherent로 carveouts 주소를 할당합니다. (dynamic allocation)

5. rproc_actuate()는 리소스 처리가 끝나면 rproc_start_subdevices()를 호출합니다.
   rproc_start_subdevices() -> rproc_vdev_do_start(): RSC_VDEV 리소스를 통해 생성된
   각 rproc_vdev의 start 함수를 호출합니다.

6. rproc_vdev_do_start() -> rproc_add_virtio_dev(): rproc_vdev 정보를 바탕으로
   struct virtio_device를 생성하고 virtio 버스에 등록합니다.
   virtio_device가 등록되면, virtio 버스는 이 장치와 매칭되는 드라이버를 찾습니다.

7. virtio_rpmsg_bus 드라이버는 VIRTIO_ID_RPMSG ID를 가지고 있으므로,
   이 드라이버의 probe 함수인 rpmsg_probe()가 호출됩니다.

8. virtio_find_vqs(): virtio_device로부터 virtqueue를 찾아 초기화합니다.
   이 virtqueue는 2단계에서 할당된 vring 메모리를 기반으로 동작합니다.
   rx와 tx 두 개의 virtqueue를 찾습니다.

9. 버퍼 할당: dma_alloc_coherent()를 다시 호출하여 메시지 페이로드(payload)를 담을 실제 통신 버퍼들(rbufs, sbufs)을 할당합니다.
   (vring 메모리와는 별개의 공간입니다.)
   버퍼 등록: virtqueue_add_inbuf()를 루프 안에서 호출하여 할당된 수신(rx) 버퍼들을 rvq(receive virtqueue)에 등록합니다.
   이제 원격 프로세서는 이 버퍼들에 메시지를 쓸 수 있습니다.
   virtio 장치가 통신할 준비가 되었음을 알립니다. (rsc table에 status update)

10. virtqueue_notify(): rvq에 버퍼가 준비되었음을 원격 프로세서에 kick을 보냅니다.
```
<br>

#### shared memory map 구성

shared memory 구성은 remote device의 resource table 로부터<br>
크게 떨어지지 않게 구성하였습니다.<br>
resource table의 경우, self-boot 시나리오 상, master에서 관여할 수가 없기 때문에,<br>
remote의 최종 형상에 따라 결정됩니다.<br>

<img src="/assets/images/legacy/rpmsgmmap.png" alt="rpmsgmmap" width="640" height="480"><br>

device tree는 다음과 같이 구성하였습니다.
```
// SPDX-License-Identifier: (GPL-2.0-or-later OR MIT)
/*
 * Copyright (C) 2025 Telechips Inc.
 */

#include "tcc807x-subcore-mbox.dtsi"

/ {
	/* Sub-Core(Master) <-> Main Core(Remote) */
	rpmsg_shm: shm@200c6000 {
		device_type = "memory";
		reg = <0x0 0x200c6000 0x0 0x8000>;
		status = "disabled";
	};

	tcc-rproc {
		compatible = "telechips,tcc807x-subcore-rproc";
		mboxes = <&a55sp_a55mp_mbox 0>;
		memory-region = <&rsc_table0>,
		                <&vdev0vring0>,
		                <&vdev0vring1>,
		                <&vdev0buffer>;
		status = "disabled";
	};
};

&reserved_memory {
	rsc_table0: rsc-table@200c6000 {
		no-map;
		/* Required rsc table size is 4K*/
		/*allocate an extra 4K margin for offset handling. */
		reg = <0x0 0x200c6000 0x0 0x2000>;
		tcc,rsc-offset = <0x3b8>;
		tcc,rsc-size = <0x1000>;
		status = "disabled";
	};

	vdev0vring0: vdev0vring0@200c8000 {
		no-map;
		reg = <0x0 0x200c8000 0x0 0x1000>;
		status = "disabled";
	};

	vdev0vring1: vdev0vring1@200c9000 {
		no-map;
		reg = <0x0 0x200c9000 0x0 0x1000>;
		status = "disabled";
	};

	vdev0buffer: vdev0buffer@200ca000 {
		no-map;
		reg = <0x0 0x200ca000 0x0 0x4000>;
		status = "disabled";
	};
};
```
<br>

## 정리

porting 후, linux kernel에서 제공하는 rpmsg-client-samples를 실행했습니다.<br>
메세지는 100회에서 3회로 수정하였습니다. (100회는 zephyr가 속도를 따라가지 못해 대부분의 메세지를 drop합니다.)<br>

<img src="/assets/images/legacy/rpmsg-linux.png" alt="rpmsg-linux" width="320" height="240">
<img src="/assets/images/legacy/rpmsg-zephyr.png" alt="rpmsg-zephyr" width="320" height="240"><br>


ipc를 rpmsg를 포팅하여 시도해 본것은 처음인데, 잘 되었습니다.<br>
rpmsg/virtio 스터디, 타 업무 진행 포함 포팅하는데 3주 정도 소요 된 것 같습니다.<br>
<br>
zephyr 빌드의 경우 주제가 달라 따로 포스팅 하지 않았습니다.<br>
