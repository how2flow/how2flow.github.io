---
permalink: /posts/kernel-build/
title: "[Linux] 커널 빌드 & 설치 (with 라즈베리파이 4)"
excerpt: "리눅스 커널 빌드 (with rpi4)"
header:
  teaser: /assets/posts/images/raspberrypi.png
categories:
  - Linux
tags:
  - kernel
  - kernel build
  - raspberry
  - raspberry pi
  - rpi
  - rpi4
toc: true
---

간단한 것부터 하나씩 포스팅 해보려고 노력중입니다..<br>
github page는 관리가 좀 귀찮은 단점이 있네요.<br>
[Documents](/documents/linux/kernel-build/)에도 업로드 예정입니다.<br>

## 리눅스 커널 빌드 및 설치

커널 버전: <span style="{{ site.code }}">4.19.y</span> 이상 (패키지 호환성)<br>

커널을 빌드하는 이유는 여러가지 있겠지만,<br>
보통은 커널에서 어떤 부분을 수정하거나 추가한 후<br>
변경사항을 테스트하고 적용하기 위해 빌드합니다.<br>

커널 빌드하는 방법 2가지를 정리해 보려고 합니다.<br>
사실 둘 다 make 입니다. 패키지는 make 빌드를 스크립트로 작성하고,<br>
대화형으로 동작하게 만들었습니다.<br>

예시는 <span style="{{ site.code}}">raspberry Pi 4</span> , <span style="{{ site.code }}">5.4.y</span> 커널입니다.<br>

<p align="center">
  <img src="/assets/posts/images/raspberrypi.png" alt="raspberrypi" width="640" height="480"><br>
  <span style="{{ site.img }}">raspberry Pi 4</span>
</p>
<br>


### Native

타겟에서 직접 소스를 받고, 빌드하고, install 하는 과정입니다.<br>

#### make 명령으로 설치하기

raspberry Pi 4 커널 5.4.y 빌드 & 설치 하는 과정입니다.<br>
raspberry Pi 4 타겟에서 직접 진행합니다.<br><br>

의존성 패키지 설치
```
$ sudo apt-get install build-essential bc bison flex libssl-dev
```

소스파일 다운로드
```
$ git clone --depth=1 -b rpi-5.4.y https://github.com/raspberrypi/linux.git
$ cd linux
$ make bcm2711_defconfig
$ make -j4
$ make modules_install dtbs_install
$ sudo cp arch/arm64/boot/Image /boot/kernel8.img
$ sudo vi /boot/config.txt
```
```
# device_tree=dtbs/5.4.47-v8+/broadcom/bcm2711-rpi-4-b.dtb
# overlay_prefix=dtbs/5.4.47-v8+/overlays/
# kernel=kernel8.img
```
```
$ reboot
```

#### 패키지로 설치하기

라즈비안 이미지(distro: <span style="{{ site.code }}">buster</span>)는 지원하지 않습니다. (2023-05-15 기준)<br><br>

우분투 이미지는 20.04, 22.04 지원됩니다. (2023-05-15 기준)<br>
아래는 우분투 이미지에서 진행과정입니다.<br>
우분투 공식 패키지가 아닌 개인 아카이브에서 패키지를 가져올 것입니다.<br><br>

의존성 패키지 설치
```
$ sudo apt-get install software-properties-common
```

PPA 추가
```
$ sudo add-apt-repository ppa:how2flow/ppa
$ sudo apt-get update
```

패키지 설치
```
$ sudo apt-get install kernel-scripts
```

소스파일 다운로드
```
$ git clone --depth=1 -b rpi-5.4.y https://github.com/raspberrypi/linux.git
```

커널 빌드 & 설치
```
$ cd linux
$ kernel-script --crosscompile

[kernel build architecture]
Input architecture: arm64

[kernel build config]
Input config: bcm2711_defconfig

[kernel build target]
Input target: all
```

커널 빌드 output 파일들이 <span style="{{ site.code }}">out/</span> 에 있습니다.<br>
```
$ cd out
$ make modules_install
$ make dtbs_install
$ cp arch/arm64/boot/Image /boot/kernel8.img
$ sudo vi /boot/config.txt
```
```
# device_tree=dtbs/5.4.47-v8+/broadcom/bcm2711-rpi-4-b.dtb
# overlay_prefix=dtbs/5.4.47-v8+/overlays/
# kernel=kernel8.img
```
```
$ reboot
```

### Cross-compile

크로스-컴파일 빌드를 하는 이유는<br>
커널 빌드는 시간이 오래 걸리기도 하고 <span style="{{ site.code }}">native_build</span>를 했을 때<br>
잘못하면 커널 시스템을 망가트릴 수 있으므로, 빌드머신을 따로 두고<br>
<span style="{{ site.code }}">cross-compile</span> 하는게 좋습니다.<br>

빌드 머신은 <span style="{{ site.code }}">x86</span> 머신, 그리고 OS는 Ubuntu 20.04 LTS 기준입니다.<br>
라즈베리파이 이미지가 설치된 메모리 장치(sd-card)를 빌드머신에 연결하고 진행합니다.<br>

#### make 명령으로 설치하기

과정은 전체적으로 Native와 비슷합니다.<br><br>

의존성 패키지 설치
```
$ sudo apt-get install build-essential bc bison flex libssl-dev gcc-aarch64-linux-gnu
```

소스파일 다운로드
```
$ git clone --depth=1 -b rpi-5.4.y https://github.com/raspberrypi/linux.git
```

먼저 커널 소스트리의 루트 경로로 이동합니다.
```
$ cd linux
```

빌드 타겟은 크게 3가지가 있는데,<br>
커널 이미지(<span style="{{ site.code }}">zImage/Image.gz (.o)</span>), 커널 모듈(<span style="{{ site.code }}">.ko</span>), 커널 디바이스트리(<span style="{{ site.code }}">.dtb</span>) 입니다.<br>

빌드시에 필요한 환경변수가 몇가지 있습니다.<br>
<span style="{{ site.code }}">ARCH</span>,
<span style="{{ site.code }}">CROSS_COMPILE</span>,
<span style="{{ site.code }}">INSTALL_MOD_PATH</span>,
<span style="{{ site.code }}">INSTALL_DTBS_PATH</span><br>

<span style="{{ site.code }}">ARCH</span>는 타겟의 아키텍쳐를 지정합니다.<br>

지정 가능한 아키텍쳐는 커널소스 루트에서
```
$ ls -lh arch/
-rw-r--r--   1 user user  31K May  8 09:51 Kconfig
drwxr-xr-x   9 user user 4.0K May  8 09:51 alpha
drwxr-xr-x  14 user user 4.0K May  8 09:51 arc
drwxr-xr-x 101 user user 4.0K May  8 09:51 arm
drwxr-xr-x  12 user user 4.0K May  8 09:51 arm64
drwxr-xr-x   9 user user 4.0K May  8 09:51 c6x
drwxr-xr-x   8 user user 4.0K May  8 09:51 h8300
drwxr-xr-x   7 user user 4.0K May  8 09:51 hexagon
drwxr-xr-x  14 user user 4.0K May  8 09:51 ia64
drwxr-xr-x  25 user user 4.0K May  8 09:51 m68k
drwxr-xr-x  10 user user 4.0K May  8 09:51 microblaze
drwxr-xr-x  52 user user 4.0K May  8 09:51 mips
drwxr-xr-x   8 user user 4.0K May  8 09:51 nds32
drwxr-xr-x   9 user user 4.0K May  8 09:51 nios2
drwxr-xr-x   8 user user 4.0K May  8 09:51 openrisc
drwxr-xr-x  10 user user 4.0K May  8 09:51 parisc
drwxr-xr-x  19 user user 4.0K May  8 09:51 powerpc
drwxr-xr-x   7 user user 4.0K May  8 09:51 riscv
drwxr-xr-x  19 user user 4.0K May  8 09:51 s390
drwxr-xr-x  15 user user 4.0K May  8 09:51 sh
drwxr-xr-x  15 user user 4.0K May  8 09:51 sparc
drwxr-xr-x   8 user user 4.0K May  8 09:51 um
drwxr-xr-x   8 user user 4.0K May  8 09:51 unicore32
drwxr-xr-x  27 user user 4.0K May  8 09:51 x86
drwxr-xr-x  11 user user 4.0K May  8 09:51 xtensa
```
Kconfig를 제외한 나머지 디렉토리 이름과 같습니다.<br><br>

<span style="{{ site.code }}">CROSS_COMPILE</span>은 빌드에 사용할 툴체인을 정합니다.<br>
타겟의 아키텍쳐에 따라 <span style=" {{ site.code }}">x86_64-linux-gnu-</span> 또는,<br>
<span style=" {{ site.code }}">aarch64-linux-gnu-</span> 등 아키텍처에 맞는 툴체인을 정하면 됩니다.<br>
예시는 <span style="{{ site.code }}">raspberry Pi 4</span> 이므로 컴파일러는 <span style=" {{ site.code }}">aarch64-linux-gnu-</span>를 사용하면 됩니다.<br>

<span style="{{ site.code }}">INSTALL_MOD_PATH</span> 와 <span style="{{ site.code }}">INSTALL_DTBS_PATH</span> 는 메모리 디바이스를 연결 했을 때 마운트 되는 장치의 부트 파티션 경로를 지정하면 됩니다.<br>
마운트 경로는 따로 지정하지 않았다면 보통 <span style="{{ site.code}}">/media/$(whoami)/..</span> 에 마운트 됩니다.<br>

만약 <span style="{{ site.code }}">sudo</span> 를 사용하면 root의 변수를 가져오기 때문에 일반 유저의 변수와 다릅니다.<br>
섞어 쓰지 마세요.<br><br>

커널 빌드하기
```
$ export ARCH=arm64
$ export CROSS_COMPILE=aarch64-linux-gnu-
$ make bcm2711_defconfig
$ make -j$(nproc)
```

커널 설치하기
```
$ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=/media/$(whoami)/\"root_partition\"" modules_install
$ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_DTBS_PATH=/media/$(whoami)/"\root_partition\"" dtbs_install
$ cp arch/arm64/boot/Image /media/$(whoami)/"boot_partition"/kernel8.img
$ vi /media/$(whoami)/"boot_partition"/config.txt
```
```
# device_tree=dtbs/5.4.47-v8+/broadcom/bcm2711-rpi-4-b.dtb
# overlay_prefix=dtbs/5.4.47-v8+/overlays/
# kernel=kernel8.img
```

#### 패키지로 설치하기

우분투 공식 패키지가 아닌 개인 아카이브에서 패키지를 가져올 것입니다.<br><br>

의존성 패키지 설치
```
$ sudo apt-get install software-properties-common
```

PPA 추가
```
$ sudo add-apt-repository ppa:how2flow/ppa
$ sudo apt-get update
```

패키지 설치
```
$ sudo apt-get install kernel-scripts
```

소스파일 다운로드
```
$ git clone --depth=1 -b rpi-5.4.y https://github.com/raspberrypi/linux.git
```

커널 빌드 & 설치
```
$ cd linux
$ kernel-script --crosscompile

[kernel build architecture]
e.g 'arm64' or 'x86' or ...
Input architecture: arm64

[kernel build config]
e.g 'i386_defconfig' or 'oldconfig' or ...
Input config: bcm2711_defconfig

[kernel build target]
e.g 'dtbs' or 'modules' or 'vmlinux' or ...
Input target: all

Would you want to separate the result files? [Y/n]: n
```

마지막 질문의 default는 Y 입니다.<br>
y/Y 선택하면 빌드 output 파일들이 <span style="{{ site.code }}">out/</span> 에 생성됩니다.<br>
n/N 선택하면 소스파일들과 같은 경로에 빌드 output 파일들이 생성됩니다.<br>
