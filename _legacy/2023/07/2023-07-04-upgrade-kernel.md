---
permalink: /legacy/upgrade-kernel/
title: "[Linux] 커널 버전 업그레이드 1(with ODROID-M1)"
excerpt: "ODROID -M1의 커널 버전을 4.19.y에서 5.10.y로 업그레이드 하는 과정입니다."
header:
  teaser: /assets/images/legacy/m1-kernelupgrade.png
categories:
  - Linux
tags:
  - 4.19.y
  - 5.10.y
  - kernel
  - kernel merge
  - kernel rebase
  - kernel upgrade
  - rebase
  - rebase --onto
  - rebase -i
  - rebase interactive
  - rebase onto
toc: true
---

오드로이드 M1 커널 버전 업그레이드 포스팅입니다.<br>
포스팅 날짜 기준으로 현재 ODROID-M1은 linux kernel 4.19 version을 지원합니다.<br>
<span style="{{ site.code }}">linux-image-4.19.219-odroid-arm64</span><br>

이번에 최신 bsp가 업데이트 되어서, 커널을 업그레이드 하게 되었습니다.<br>
아직 공식 릴리즈가 되지 않았기 때문에 일부 과정만 올라갑니다.<br>

다음을 준비합니다.
```
- PC (Ubuntu 20.04LTS or later)
- ODROID-M1 board (x1)
- ssh 사용이 가능한 환경
```

## 소스코드/커밋 준비하기

리눅스 커널 소스는 특히 bsp 같은 경우는, 버전을 업그레이드 할 때,<br>
기존에 지원한 기능들을 그대로 지원하기 위해서 커밋들을 통으로 새 버전에 옮겨야 합니다.<br>

커밋 개수도 많고, 다른 사람들과 협업하기 때문에, 중구난방한 커밋들 정리가 필요합니다.<br>
또한, 다른 사람이 만든 커밋은 내가 정리하고 테스트하는 것보다 커밋을 만든 사람이 하는게 더 효율적입니다.<br>

그래서 사전에 커밋을 정리해야 합니다.<br>

### 소스코드 다운로드

깃 명령으로 커널소스를 다운로드 합니다.<br>

```
$ git clone -b odroidm1-4.19.y --single-branch https://github.com/hardkernel/linux
```

### 원본 로그 백업하기

커밋을 통으로 옮기기 전에 커밋들을 한번 정리해야 하고,<br>
정리하는 과정 중에서 원래 커밋과 쉽게 비교하기 위해 백업합니다.<br>

로그를 확인합니다.
```
$ cd linux
$ git log
commit 89b88ec44cb98ec17b8460cc3c0127fcb1f7b159 (HEAD -> odroidm1-4.19.y
, origin/odroidm1-4.19.y)
Author: Steve Jeong <how2soft@gmail.com>
Date:   Wed May 24 11:58:12 2023 +0900

    ODROID-M1: dtb/dtbo: Update can.
    
    Revert CAN driver CAN to CANFD
    Change can-clock-rate 300M to 200M
    
    Signed-off-by: Steve Jeong <how2soft@gmail.com>
    Change-Id: I5dc8da36ba2e15065b4580b8eab21fde104ccfdc

commit c4b1b4fb2a77703b3c53f8ac8dd723b300371497
Author: Luke go <sangch.go@gmail.com>
Date:   Wed May 17 16:35:33 2023 +0900

    ODROID-M1: dtb/dtbo: pcf8563 is added.
    
    Signed-off-by: Luke go <sangch.go@gmail.com>
    Signed-off-by: Steve Jeong <how2soft@gmail.com>
    Change-Id: Ie4757602bcbdf9988cd171e984b38fd3853d8499

commit f4c79b0ce3deada3dd98f1eeb79f62c5b9f4da6a
Author: Steve Jeong <how2soft@gmail.com>
```

<br>
현재 최신 커밋을 기준으로 업그레이드를 진행합니다.<br>
한눈에 보기 어렵기 때문에 <span style="{{ site.code }}">--oneline</span> 옵션을 사용합니다.
```
$ git log --oneline
89b88ec44cb9 (HEAD -> odroidm1-4.19.y, origin/odroidm1-4.19.y                                                                                                                                        [1/57]
) ODROID-M1: dtb/dtbo: Update can.                                                                                                                                                                         
c4b1b4fb2a77 ODROID-M1: dtb/dtbo: pcf8563 is added.
...

```

<br>
이제부터 <span style="{{ site.code }}">dffdb03eb14f</span> 부터 <span style="{{ site.code }}">89b88ec44cb9</span> 까지 커밋들을 정리하겠습니다.<br>
저는 이 로그들을 텍스트 파일로 따로 저장해 둡니다.
```
$ git log --oneline > ~/odroidm1-4.19
```

<br>
나중에 원본의 로그를 편하게 보기 위함입니다.<br>
물론 <span style="{{ site.code }}">dffdb03eb14f</span> 아래는 저장하지 않습니다.<br>

### 작업 브렌치 만들기

이제 커밋을 정리할 작업용 브렌치를 만듭니다.
```
$ git checkout -b odroidm1-5.10.y-dev
```

<br>
전체적인 작업 과정은 <span style="{{ site.code }}">rebase</span> -> <span style="{{ site.code }}">diff</span> 입니다.<br>

결과적으로, 원본 브렌치와 작업 브렌치의 소스가 일치해야 합니다.<br>

### Rebase 시작하기 전

커밋을 정리해야 하는 구간을 알았으니까 <span style="{{ site.code }}">rebase</span> 를 사용해서 커밋을 정리합니다.<br>
리베이스 범위는 내가 원하는 구간 바로 이전 커밋을 파라미터로 넘겨주면 됩니다.<br>

<span style="{{ site.code }}">rebase</span>는 <span style="{{ site.code }}">merge</span> 처럼 커밋 히스토리를 정리할 때 사용합니다.<br>
<span style="{{ site.code }}">rebase</span>가 히스토리는 깔끔하게 정리되기 때문에, 보통은 <span style="{{ site.code }}">rebase</span> 를 많이 사용합니다.<br>

대량의 커밋을 편집/리베이스 하기때문에 <span style="{{ site.code }}">-i</span> ( <span style="{{ site.code }}">--interactive</span> ) 옵션을 사용합니다.<br>

rebase interactive 모드에 진입한다고 하면 다음 명령어를 사용한다고 이해하면 됩니다.<br>
```
$ git rebase -i 863e1d0fec38
```
```
pick dffdb03eb14f ODROID-M1: arch/arm64: add new board Hardkernel's ODROID-M1
pick af4f46884ced ODROID-M1: dts/dtbo: Introduce device tree overlay

...

pick c4b1b4fb2a77 ODROID-M1: dtb/dtbo: pcf8563 is added.
pick 89b88ec44cb9 ODROID-M1: dtb/dtbo: Update can.
```

<br>
rebase를 실행하게 되면 위 화면을 볼 수 있습니다.<br>
위로 갈수록 예전 커밋이고, 아래로 갈수록 최신 커밋입니다.<br>

저는 M1 제품에서 <span style="{{ site.code }}">I/O peripherals</span> 와 <span style="{{ site.code }}">NPU</span> 를 담당했기 때문에<br>
이 2가지를 제외한 나머지 커밋들은 최신으로 보낼 것입니다.<br>
위치를 바꾸는 이유는 커밋을 정리하고 나면 새로운 bsp위에 커밋 하나씩 <span style="{{ site.code }}">cherry-pick</span> 해서 테스트 할 예정이기 때문입니다.<br>
제가 커밋을 정리하고 다른 사람들에게 넘겨줘야 하기 때문에 제 담당 커밋을 먼저 올릴 수 있도록 바꾸는 것입니다.<br>

#### 권장사항

1. 커밋을 옮길때 주의할 점은 기존의 버전 순서를 최대한 지키는 방향으로,<br>
2. 옮기는 커밋의 수는 최대한 적게, 3. 옮기는 위치도 최대한 가까이 합니다.<br>

무슨 의미나면,<br>

1번 기존의 버전 순서를 최대한 지키는 것은 다음과 같습니다.<br>
커밋의 순서가 다음과 같을 때,
```
a
b
c
d
e
f
g
root
```

b 와 c 커밋을 e커밋 이전으로 옮기려고 한다면
```
a
d
e
b
c
f
g
root
```
이렇게 옮깁니다.<br>

<br>
상황에 따라서 이와 다르게 해야 하는 경우도 있습니다만,<br>
가능한 피하는게 좋습니다.<br>

2번과 3번은 쉽게 말해서 적은 양의 커밋을 조금씩만 옮기라는 뜻입니다.<br>
쓰다보니 서론이 너무 길어져서 대충 생략하겠습니다.<br>

## Git Rebase

### 담당 아닌(카메라) 커밋 재배치 (1)

먼저 카메라 관련 커밋부터 최신으로 올리겠습니다.<br>
관련 키워드는 <span style="{{ site.code }}">ov5647</span> , <span style="{{ site.code }}">imx219</span>, <span style="{{ site.code }}">imx477</span> 입니다.<br>
rebase는 A 커밋을 B 아래로 배치하던지 , B커밋을 A 위로 배치하던지 결과는 같습니다.<br>
정답은 없지만 어떻게 배치하냐에 따라 conflict를 해결하는 난이도가 크게 달라질 수 있습니다.<br>

<span style="{{ site.code }}">237a94fde037</span> 커밋 하나를 최신 커밋으로 올려보겠습니다.
```
...

pick c4b1b4fb2a77 ODROID-M1: dtb/dtbo: pcf8563 is added.
pick 89b88ec44cb9 ODROID-M1: dtb/dtbo: Update can.

pick 237a94fde037 ODROID-M1: ov5647: Code Refactory.
```
저는 에디터가 <span style="{{ site.code }}">vi</span> 이므로,
```
:wq
```

<br>
저장하고 나와 줍니다.<br>

특별한 conflict가 없으면 이렇게 나타납니다.
```
Successfully rebased and updated detached HEAD.
```

<br>
로그를 확인해보면 최신 커밋이 바뀌어 있습니다.
```
$ git log --oneline
48cc4584feb9 (HEAD -> odroidm1-5.10.y-dev) ODROID-M1: ov5647: Code
Refactory.
83f53b608f01 ODROID-M1: dtb/dtbo: Update can.
c80637697875 ODROID-M1: dtb/dtbo: pcf8563 is added.

...
```

<br>
rebase가 완료되면 원본 브렌치와 비교합니다.
```
$ git diff odroidm1-4.19.y
```

<br>
차이가 없으면 계속 진행합니다.<br>
즉, <span style="{{ site.code }}">git rebase -i 863e1d0fec38</span> -> rebase 완료 -> <span style="{{ site.code }}"> git diff odroidm1-4.19.y</span><br>
정리가 끝날 때 까지 무한 반복입니다.<br>

<br>
이런 과정을 계속 반복해서 커밋을 옮겨보겠습니다.
```
pick c80637697875 ODROID-M1: dtb/dtbo: pcf8563 is added.
pick 83f53b608f01 ODROID-M1: dtb/dtbo: Update can.

pick c3d55cfa3b72 ODROID-M1: ov5647: Support v4l2 features.
pick 48cc4584feb9 ODROID-M1: ov5647: Code Refactory.
```

...
```
pick a01b7d12f5a1 ODROID-M1: dtb/dtbo: pcf8563 is added.
pick 003dbe457a48 ODROID-M1: dtb/dtbo: Update can.

pick 56cacc7b30e1 ODROID-M1: ov5647: Support flip feature.
pick d06e325696ed ODROID-M1: ov5647: Support test pattern feature.
pick 4823899dc826 ODROID-M1: ov5647: changed color format x8 to x10.

pick a3ca322e0b20 ODROID-M1: ov5647: Support v4l2 features.
pick eab7a8521bcf ODROID-M1: ov5647: Code Refactory.
```

<br>
여기까지 했다면 로그가 다음과 같습니다.
```
$ git log --oneline
2aaaf4755b0c (HEAD -> odroidm1-5.10.y-dev) ODROID-M1: ov5647: Code 
Refactory.
17dc84e453de ODROID-M1: ov5647: Support v4l2 features.
2a7b1d1ef22c ODROID-M1: ov5647: changed color format x8 to x10.
9ce0c9b36983 ODROID-M1: ov5647: Support test pattern feature.
91e8fa9f3f93 ODROID-M1: ov5647: Support flip feature.
1daa7f3499d8 ODROID-M1: dtb/dtbo: Update can.
066cbd50fb90 ODROID-M1: dtb/dtbo: pcf8563 is added.
...
```

<br>
별다른 conflict가 없어서, 크게 수정을 해보겠습니다.<br>
rebase를 다음과 같이 진행했습니다.
```
pick b14bd90260a0 ODROID-M1: dtb/dtbo: change can0 clock freq 150MHz to 300MHz
pick 066cbd50fb90 ODROID-M1: dtb/dtbo: pcf8563 is added.
pick 1daa7f3499d8 ODROID-M1: dtb/dtbo: Update can.

pick 9d269d57ee49 ODROID: ov5647: change power gpio setting.
pick 91e8fa9f3f93 ODROID-M1: ov5647: Support flip feature.
pick 9ce0c9b36983 ODROID-M1: ov5647: Support test pattern feature.
pick 2a7b1d1ef22c ODROID-M1: ov5647: changed color format x8 to x10.
pick 17dc84e453de ODROID-M1: ov5647: Support v4l2 features.
pick 2aaaf4755b0c ODROID-M1: ov5647: Code Refactory.
```

<br>
한참 이전에 있던 <span style="{{ site.code }}">9d269d57ee49</span> 커밋을 최신 커밋에 가깝게 pick 했습니다.<br>
~~그런데 세상에 conflict가 없네요~~

계속 진행합니다.
```
...

pick 6cd3d1671793 ODROID-M1: dtb/dtbo: pcf8563 is added.
pick e7578da29066 ODROID-M1: dtb/dtbo: Update can.

pick 92fe4ae26a26 ODROID: dts/dtbo: Add OV5647 overlays.
pick 89bb9b887bb4 ODROID: ov5647: change power gpio setting.

...
```

<br>
여기서 conflict가 발생했습니다.
```
Auto-merging arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
CONFLICT (content): Merge conflict in arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
error: could not apply c0286e3c1f3c... ODROID-M1: arm64/configs: Supported IMX219.
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply c0286e3c1f3c... ODROID-M1: arm64/configs: Supported IMX219.
```

<br>
rebase 과정중에 <span style="{{ site.code }}">c0286e3c1f3c</span> 커밋과 conflict가 발생했습니다.<br>
어디서 충돌났는지 확인합니다.<br>
```
$ git diff
diff --cc arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
index 4c481d2cf108,456f8233723f..000000000000
--- a/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
+++ b/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
@@@ -13,7 -13,9 +13,13 @@@ dtbo-$(CONFIG_ARCH_ROCKCHIP_ODROIDM1) +
        spi0.dtbo \
        uart0-with-ctsrts.dtbo \
        uart0.dtbo \
++<<<<<<< HEAD
 +      uart1.dtbo
++=======
+       uart1.dtbo \
+       ov5647.dtbo \
+       imx219.dtbo
++>>>>>>> c0286e3c1f3c (ODROID-M1: arm64/configs: Supported IMX219.)
  
  targets += $(dtbo-y)
  always := $(dtbo-y)
```
<span style="{{ site.code }}">arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile</span> 에서 conflict가 발생했습니다.<br>

<br>
문제의 커밋을 확인합니다.
```
$ git show c0286e3c1f3c
commit c0286e3c1f3c11aa932c1b362e686ddd9858e989
Author: Luke go <sangch.go@gmail.com>
Date:   Mon Dec 13 12:17:24 2021 +0900

    ODROID-M1: arm64/configs: Supported IMX219.

    - Add dtbo feature.
    - Removed unused test pattern menus.
    - ioctl: Add hdr cfg commands and Removed awb cfg commands.
    - Added mutex.

    Change-Id: I1f97d956a09ddeaebc5ef3cc158560a18821f7ff
    Signed-off-by: Luke go <sangch.go@gmail.com>

diff --git a/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile b/arch/arm64/boot/dts/rockchi
p/overlays/odroidm1/Makefile
index 466970be96ff..456f8233723f 100644
--- a/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
+++ b/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
@@ -14,7 +14,8 @@ dtbo-$(CONFIG_ARCH_ROCKCHIP_ODROIDM1) += \
        uart0-with-ctsrts.dtbo \
        uart0.dtbo \
        uart1.dtbo \
-       ov5647.dtbo
+       ov5647.dtbo \
+       imx219.dtbo
```

<br>
기존 파일에서 'imx219.dtbo'가 추가되는 형태입니다.<br>
다만 rebase를 통해서 parents 커밋이 변경되었기 때문에, 'ov5647.dtbo'는 존재하면 안됩니다.<br>
uart1.dtbo와 imx219.dtbo를 남기고 ov5647.dtbo를 삭제해야 합니다.<br>

<br>
문서를 수정한 후, 변경된 부분을 확인합니다.
```
diff --cc arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
index 4c481d2cf108,456f8233723f..000000000000
--- a/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
+++ b/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
@@@ -13,7 -13,9 +13,8 @@@ dtbo-$(CONFIG_ARCH_ROCKCHIP_ODROIDM1) +
        spi0.dtbo \
        uart0-with-ctsrts.dtbo \
        uart0.dtbo \
-       uart1.dtbo
+       uart1.dtbo \
 -      ov5647.dtbo \
+       imx219.dtbo
```

<br>
변경이 끝났으면 변경된 사항을 반영하고, <span style="{{ site.code }}">git rebase --continue</span> 명령으로 rebase를 계속 진행합니다.
```
$ git add arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
$ git rebase --continue
```

<br>
rebase를 하면서 conflict가 계속 발생할 수 있습니다.<br>
그래서 처음 개발할 때 커밋을 올릴 때 올릴 내용을 잘 고려해야 합니다.<br>
~~범인되기 싫으면~~

<br>
conflict를 또 만났습니다.
```
[detached HEAD c4ef1203d280] ODROID-M1: arm64/configs: Supported IMX219.
 Author: Luke go <sangch.go@gmail.com>
 4 files changed, 341 insertions(+), 108 deletions(-)
 create mode 100644 arch/arm64/boot/dts/rockchip/overlays/odroidm1/imx219.dts
Auto-merging arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
CONFLICT (content): Merge conflict in arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
error: could not apply bcd50fe03a4b... ODROID: imx477: dts/overlay: Add device tree overlay.
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply bcd50fe03a4b... ODROID: imx477: dts/overlay: Add device tree overlay.
```

<br>
이번에도 파일을 확인합니다.
```
$ git diff
        uart0-with-ctsrts.dtbo \
        uart0.dtbo \
        uart1.dtbo \
++<<<<<<< HEAD
 +      imx219.dtbo
++=======
+       ov5647.dtbo \
+       imx219.dtbo \
+       imx477.dtbo
++>>>>>>> bcd50fe03a4b (ODROID: imx477: dts/overlay: Add device tree overlay.)
```

<br>
이전 conflict와 동일한 원인입니다.<br>
HEAD와 충돌난 <span style="{{ site.code }}">bcd50fe03a4b</span> 을 확인하고, 수정합니다.<br>
전과 동일하게 ov5647.dtbo를 제거하고 진행했습니다.<br>

<br>
이런, 또 conflict가 발생했습니다.<br>
그런데 이번에는 좀 다릅니다.
```
diff --cc arch/arm64/boot/dts/rockchip/rk3568-odroid.dtsi
index e0aa396aa4de,688d2f621380..000000000000
--- a/arch/arm64/boot/dts/rockchip/rk3568-odroid.dtsi
+++ b/arch/arm64/boot/dts/rockchip/rk3568-odroid.dtsi
@@@ -70,6 -70,18 +70,21 @@@
        /delete-node/ gt1x@14;
  };

++<<<<<<< HEAD
++=======
+ &i2c2 {
+       status = "disabled";
+       pinctrl-names = "default";
+       pinctrl-0 = <&i2c2m1_xfer>;
+ };
+
+ &i2c3 {
+       status = "disabled";
+       pinctrl-names = "default";
+       pinctrl-0 = <&i2c3m1_xfer>;
+ };
+
++>>>>>>> d77837fb5910 (ODROID-M1: dts/dtbo: modify i2c0 overlay)
```

<br>
역시 동일한 방식으로 <span style="{{ site.code }}">d77837fb5910</span> 커밋 내용을 확인해서 변경합니다.
```
$ git show d77837fb5910

...

diff --git a/arch/arm64/boot/dts/rockchip/rk3568-odroid.dtsi b/arch/arm64/boot/dts/rockchip/rk3568
-odroid.dtsi
index 7c69507c500f..688d2f621380 100644
--- a/arch/arm64/boot/dts/rockchip/rk3568-odroid.dtsi
+++ b/arch/arm64/boot/dts/rockchip/rk3568-odroid.dtsi
@@ -76,6 +76,12 @@
        pinctrl-0 = <&i2c2m1_xfer>;
 };
 
+&i2c3 {
+       status = "disabled";
+       pinctrl-names = "default";
+       pinctrl-0 = <&i2c3m1_xfer>;
+};
+
```

<br>
왜 이런 conflict가 발생했을까요?<br>
아마 지금 옮기고 있는 <span style="{{ site.code }}">92fe4ae26a26</span> 코드에서 i2c2내용이 포함되어 있을 것입니다.
```
$ git show 92fe4ae26a26
diff --git a/arch/arm64/boot/dts/rockchip/rk3568-odroid.dtsi b/arch/arm64/boot/dts/rockchip/rk3568-odroid.dtsi
index 2de3981a0377..f16b7b91d8dd 100644
--- a/arch/arm64/boot/dts/rockchip/rk3568-odroid.dtsi
+++ b/arch/arm64/boot/dts/rockchip/rk3568-odroid.dtsi
@@ -70,6 +70,12 @@
        /delete-node/ gt1x@14;
 };

+&i2c2 {
+       status = "disabled";
+       pinctrl-names = "default";
+       pinctrl-0 = <&i2c2m1_xfer>;
+};
+
```

<br>
예상이 맞았습니다.<br>
rebase를 하면서 parents커밋에 i2c2 내용이 없기때문에 발생한 conflict입니다.<br>
i2c2 부분을 날리고 계속 진행합니다.
```
$ git add arch/arm64/boot/dts/rockchip/rk3568-odroid.dtsi
$ git rebase --continue
```

<br>
쭉쭉 진행하다가 엄청난 녀석을 만났습니다..
```
commit c3e399f5853b780a41090fae5cd068b9c23bf591
Author: steve.jeong <jkhpro1003@gmail.com>
Date:   Fri Mar 18 17:11:48 2022 +0900

    ODROID-M1: dtb/dtbo: Add can0 overlay
    
    Signed-off-by: steve.jeong <jkhpro1003@gmail.com>
    Change-Id: Icab5a5fd350537e9e9f80c13b6e92925a598c86f

diff --git a/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile b/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
index cfadd34bf3c4..b8549d9f10a0 100644
--- a/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
+++ b/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
@@ -1,21 +1,22 @@
 # Overlays for the Odroid platform
 
 dtbo-$(CONFIG_ARCH_ROCKCHIP_ODROIDM1) += \
+       can0.dtbo \
        display_vu8m.dtbo \
        hktft32.dtbo \
        i2c0.dtbo \
        i2c1.dtbo \
+       imx219.dtbo \
+       imx477.dtbo \
+       ov5647.dtbo \
        pwm1.dtbo \
        pwm2.dtbo \
        pwm9.dtbo \
+       rknpu.dtbo \
        spi0.dtbo \
        uart0-with-ctsrts.dtbo \
        uart0.dtbo \
-       uart1.dtbo \
-       ov5647.dtbo \
-       imx219.dtbo \
-       imx477.dtbo \
-       rknpu.dtbo
+       uart1.dtbo
 
 targets += $(dtbo-y)
 always := $(dtbo-y)
diff --git a/arch/arm64/boot/dts/rockchip/overlays/odroidm1/can0.dts b/arch/arm64/boot/dts/rockchip/overlays/odroidm1/can0.dts
new file mode 100644
index 000000000000..cf3b2b07ace9
--- /dev/null
+++ b/arch/arm64/boot/dts/rockchip/overlays/odroidm1/can0.dts
```

<br>
굉장히 더러운 커밋이어서 대체 누구지? 하고 ~~범인~~ 저자를 찾았습니다.<br>
~~근데 나잖아??~~<br>

과거의 기억이 스쳐 지나가네요..<br>
Makefile에 있는 dtbo 파일들을 알파벳 순서로 바꿔달라는 요청이 있었는데<br>
can dtbo 커밋에다가 요청사항을 그냥 반영 했었습니다.<br>

(커밋을 나눠서 올렸어야 했음.. 커밋 잘 생각해서 잘 올리자...)<br>
결국 uart1만 살리고 rebase 진행했습니다.<br>

<br>
conflict가 많이 나왔기 때문에 설레는 마음으로 원본 브렌치와 diff를 합니다.
```
$ git diff odroidm1-4.19.y
diff --git a/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile b/arch/arm64/boot/dts/rockchi
p/overlays/odroidm1/Makefile
index 8f8598584da4..c164a2119001 100644
--- a/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
+++ b/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
@@ -24,7 +24,8 @@ dtbo-$(CONFIG_ARCH_ROCKCHIP_ODROIDM1) += \
        ttyfiq0_115200.dtbo \
        uart0-with-ctsrts.dtbo \
        uart0.dtbo \
-       uart1.dtbo
+       uart1.dtbo \
+       ov5647.dtbo
```

<br>
이정도 차이는 모든 커밋의 정리가 끝나고 수정해도 됩니다.<br>
ov5647.dtbo가 마지막에 나타난 이유는 ov5647 dtbo 커밋을 최신커밋으로 보냈기 때문입니다.<br>

ov5647.dtbo가 중복으로 나타나는 이유는<br>
~~망할 저의 can0 dtbo 커밋에서 ov5647.dtbo를 포함해서 알파벳순으로 정렬을 했기 때문에..~~<br>
원본에서 ov5647.dtbo가 추가되어 있는 상태에서 can0 dtbo 커밋이 ov5647.dtbo의 위치를 변경했기 때문입니다.<br>

rebase를 하면서 ov5647.dtbo가 존재하지 않게 되었습니다. 하지만 ov5647.dtbo의 위치 변경이 can0 커밋의 히스토리에 남아있어서 발생한 문제입니다.<br>

이번에는 과감하게 2개 커밋을 같이 옮기겠습니다.<br>
타겟은 <span style="{{ site.code }}">0b9ec4615e02</span> , <span style="{{ site.code }}">43d85c065989</span> 커밋 입니다.

<br>
rebase interactive 모드로 진입합니다.
```
...

pick a09b47ed030c ODROID-M1: dtb/dtbo: pcf8563 is added.
pick cda1c5132748 ODROID-M1: dtb/dtbo: Update can.

pick 0b9ec4615e02 ODROID: ov5647: Support ov5647, default setting.
pick 43d85c065989 ODROID: ov5647: Add rockchip implementation to support android.

pick 8557240bdcaf ODROID: dts/dtbo: Add OV5647 overlays.
pick d83c96e01bcd ODROID: ov5647: change power gpio setting.

...

```

<br>
conflict 없이 rebase 되었습니다.<br>
원본 브렌치와의 차이도 문제 없습니다.<br>

<br>
rebase interactive 모드로 진입해서 지금까지 정리된 커밋들을 확인합니다.
```
...

pick 66452583e80c ODROID-M1: dtb/dtbo: pcf8563 is added.
pick 603c0a4bd475 ODROID-M1: dtb/dtbo: Update can.
pick a114b0c7fbda ODROID: ov5647: Support ov5647, default setting.
pick 6957edff1233 ODROID: ov5647: Add rockchip implementation to support android.
pick d9e3d41dbed3 ODROID: dts/dtbo: Add OV5647 overlays.
pick bcbecc4230f2 ODROID: ov5647: change power gpio setting.
pick d4e209e7d3df ODROID-M1: ov5647: Support flip feature.
pick 4a1076d132b7 ODROID-M1: ov5647: Support test pattern feature.
pick 7799ddedeaa6 ODROID-M1: ov5647: changed color format x8 to x10.
pick f8cd5b8ccfe0 ODROID-M1: ov5647: Support v4l2 features.
pick aca1de1bc555 ODROID-M1: ov5647: Code Refactory.
```

<br>
ov5647 관련 커밋들 모두 최신 커밋으로 올리는 데 성공했습니다.<br>
이제 <span style="{{ site.code }}">imx219</span> , <span style="{{ site.code }}">imx477</span> 관련 커밋들만 올리면 됩니다.<br>

### 담당 아닌(카메라) 커밋 재배치 (2)

나머지 카메라 관련 커밋들을 올리기 위해, <span style="{{ site.code }}">ov5647</span> 과 <span style="{{ site.code }}">imx</span> 사이에 있는 엄청난 양의 defconfig와 dtbo 커밋들을 정리해 줄겁니다.<br>
이 커밋들을 한데 모아서 defconfig 커밋들은 첫번째 커밋인 <span style="{{ site.code }}">dffdb03eb14f</span>에 합치고, dtbo 파일들은 <span style="{{ site.code }}">bfb15941df3c</span> 커밋 위로 쌓아 올릴 것입니다.<br>

이렇게 하는 이유는<br>
카메라 커밋을 최신으로 올리지 않고 나머지 커밋들을 이전 커밋으로 옮기는 것 역시<br>
위에서 언급한 대로 동일한 결과를 갖습니다.<br>

defconfig는 하나의 커밋으로, dtbo는 커밋들을 유지한 채 쌓아 올리는 이유는<br> 
defconfig는 dtbo와 다르게 특별한 feature가 아닙니다.<br>

##### defconfig, dtb/dtbo 통일하기

rebase interactive 모드를 진행합니다.

제일 쉬운거 부터 하겠습니다.<br>
squash 명령으로 올라간 커밋부터 합쳐줍니다.
```
...

pick 75c997655a7b ODROID-M1: defconfig: Add mcp251x to odroidm1_defconfig.
pick a47a2bb3529d ODROID-M1: driver/i2c: Add driver "speed" attribution
s 3c1b5e7e2d98 squash! ODROID-M1: driver/i2c: Add driver "speed" attribution
pick 6649688b8939 ODROID-M1: dtb/dtbo: change can0 clock freq 150MHz to 300MHz
pick 66452583e80c ODROID-M1: dtb/dtbo: pcf8563 is added.

...
```

<br>
git 명령어로 <span style="{{ site.code }}">squash</span> 또는 <span style="{{ site.code }}">s</span>으로 커밋을 합칠 수 있습니다.<br>
커밋 메세지는 경우에 따라 부모 혹은 자식 커밋 메세지로 통일(필요시 부연설명 추가)합니다.<br>

저는 rebase interactive 모드에서 squash 명령어를 사용할 때<br>
먼저 pick으로 순서 먼저 맞추고 실행하는 것을 선호합니다.<br>

이제 defconfig를 합치겠습니다.<br>
가장 많이 몰려있는 부분에서 가장 적게 움직일만한 커밋 몇개를 우선 배치하겠습니다.<br>
rebase를 통해 조금씩 합쳐나가는 겁니다.
```
...

pick 9b9e1f00e2bf ODROID-M1: net/wireless: update rtl88xx driver, now supports 8811, 8812 and 8814
pick fcb5cb40952b ODROID-M1: dtb/dtbo: Add onewire
pick 287aced2227a ODROID-M1: defconfig: Add w1-wire (gpio)
pick d40e6d2f7a97 ODROID-M1: defconfig: Add dht11 humidity sensor
pick 683c4de9af69 ODROID-M1: defconfig/can: modify built-in to module
pick 75c997655a7b ODROID-M1: defconfig: Add mcp251x to odroidm1_defconfig.
pick f625c76a204b ODROID-M1: arm64/dts: add support 115200bps at ttyFIQ0
pick c4d79d1f90d3 ODROID-M1: dtb/dtbo: Add dht11 humidity sensor

...
```

<br>
conflict가 발생하지 않았기 때문에 하나의 커밋으로 합쳐주겠습니다.<br>
다시 rebase를 실행했을 때, hash값이 바뀌는 것은 정상입니다.
```
...

pick 9b9e1f00e2bf ODROID-M1: net/wireless: update rtl88xx driver, now supports 8811, 8812 and 8814
pick fcb5cb40952b ODROID-M1: dtb/dtbo: Add onewire
pick 287aced2227a ODROID-M1: defconfig: Add w1-wire (gpio)
s deb0cf300a34 ODROID-M1: defconfig: Add dht11 humidity sensor
s 69a6f0147b36 ODROID-M1: defconfig/can: modify built-in to module
s 9aec779604a4 ODROID-M1: defconfig: Add mcp251x to odroidm1_defconfig.
pick 4aebdb0fc348 ODROID-M1: arm64/dts: add support 115200bps at ttyFIQ0
pick 5265adbeea9c ODROID-M1: dtb/dtbo: Add dht11 humidity sensor

...
```

<br>
커밋메세지는 어차피 합칠 예정이라 대충 짓겠습니다.
```
ODROID-M1: defconfig

Signed-off-by: Steve Jeong <how2soft@gmail.com>
Change-Id: I75961449725b9cd2e1e34d275bb8be9de6574e53
```

<br>
계속 진행합니다.<br>
모으고 합치고 모으고 합치고...<br>

어느정도 진행하다 보면 conflict가 발생할 수 있습니다.<br>
conflict가 발생하는 이유는 계속 언급했듯, 커밋이 지저분 하기 때문입니다.<br>

최대한 더러운 커밋들을 피해서, 가장 이상적인 rebase 결과는 다음과 같습니다.
```
$ git log --oneline arch/arm64/configs/odroidm1_defconfig
7cbc79864dc6 ODROID: ov5647: Support ov5647, default setting.
ed31f4cc4b12 ODROID-M1: dtb/dtbo: pcf8563 is added.
dcfece3adfd9 ODROID-M1: defconfig
bedca205a0e1 ODROID-M1: arm64/configs: Supported IMX219.
8563e9a08fc2 ODROID-M1: arm64/dts: add fb_hktft32 and ads7846 dtbo for Hardkernel 3.2 inch TFT LCD
7506bad7e973 ODROID-M1: staging/fbtft: add fb_hktft32 module for Hardkernel 3.2 inch TFT LCD
dffdb03eb14f ODROID-M1: arch/arm64: add new board Hardkernel's ODROID-M1
```

<br>
위 커밋이 지저분한 이유는 defconfig 커밋인데 defconfig 파일 외에 다른 파일들도 커밋내용에 있기 때문입니다.<br>

<br>
하나씩 합쳐보겠습니다.
```
...

pick 3c71a14c3400 gpiomem driver for rk3568
pick 9e1b1581567d ODROID-M1: net/wireless: update rtl88xx driver, now supports 8811, 8812 and 8814
pick dcfece3adfd9 ODROID-M1: defconfig
pick ed31f4cc4b12 ODROID-M1: dtb/dtbo: pcf8563 is added.
pick 1276382eceb9 ODROID-M1: dtb/dtbo: Add onewire

...
```

<br>
conflict를 해결해 줍니다.<br>
만약 <span style="{{ site.code }}">overlays/odroidm1/Makefile</span> 에서 conflict가 발생한다면,<br>
dht11.dtbo를 제거하면 됩니다.<br>

diff로 원본 브렌치를 확인해보면,<br>
dht11.dtbo의 위치가 바뀌었습니다.
```
$ git diff odroidm1-4.19.y

diff --git a/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile b/arch/arm64/boot/dts/rockchi
p/overlays/odroidm1/Makefile
index 8f8598584da4..71ef40f90b11 100644
--- a/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
+++ b/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
@@ -3,8 +3,8 @@
 dtbo-$(CONFIG_ARCH_ROCKCHIP_ODROIDM1) += \
        blueled_off.dtbo \
        can0.dtbo \
-       dht11.dtbo \
        display_vu8m.dtbo \
+       dht11.dtbo \
        hktft32.dtbo \
        i2c0.dtbo \
        i2c1.dtbo \
@@ -24,7 +24,8 @@ dtbo-$(CONFIG_ARCH_ROCKCHIP_ODROIDM1) += \
        ttyfiq0_115200.dtbo \
        uart0-with-ctsrts.dtbo \
        uart0.dtbo \
-       uart1.dtbo
+       uart1.dtbo \
+       ov5647.dtbo
 
 targets += $(dtbo-y)
 always := $(dtbo-y)
```

<br>
... 이렇게 계속 진행할 수도 있지만, 더러운 커밋을 봤으면 풀어주는게 더 좋습니다.<br>
나중에 한번 더 업그레이드를 진행할 때, 똑같은 상황을 마주하게 될테니까요
```
...

pick dcfece3adfd9 ODROID-M1: defconfig
pick b245a844c752 ODROID-M1: dtb/dtbo: pcf8563 is added.

...
```

<br>
rebase를 진행한 후 다시 실행하면 hash값이 변경되는 것은 정상입니다.<br>
```
$ git show b245a844c752
diff --git a/arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile b/arch/arm64/boot/dts/rockchi
p/overlays/odroidm1/Makefile

...

diff --git a/arch/arm64/boot/dts/rockchip/overlays/odroidm1/pcf8563.dts b/arch/arm64/boot/dts/rock
chip/overlays/odroidm1/pcf8563.dts

...

diff --git a/arch/arm64/configs/odroidm1_defconfig b/arch/arm64/configs/odroidm1_defconfig
index 40c128286f4b..97f1d56d65f5 100644
--- a/arch/arm64/configs/odroidm1_defconfig
+++ b/arch/arm64/configs/odroidm1_defconfig
@@ -4233,7 +4233,7 @@ CONFIG_RTC_DRV_RK808=y
 # CONFIG_RTC_DRV_PCF8523 is not set
 # CONFIG_RTC_DRV_PCF85063 is not set
 # CONFIG_RTC_DRV_PCF85363 is not set
-# CONFIG_RTC_DRV_PCF8563 is not set
+CONFIG_RTC_DRV_PCF8563=y
 # CONFIG_RTC_DRV_PCF8583 is not set
 # CONFIG_RTC_DRV_M41T80 is not set
 # CONFIG_RTC_DRV_BQ32K is not set
```

<br>
이 커밋을 defconfig 와 defconfig가 아닌 부분, 2개로 나눠 보겠습니다.<br>
```
...

pick dcfece3adfd9 ODROID-M1: defconfig
e b245a844c752 ODROID-M1: dtb/dtbo: pcf8563 is added.

...
```

<br>
타겟 커밋은 <span style="{{ site.code }}">b245a844c752</span> 입니다.
rebase interactive 모드에서 명령어로 <span style="{{ site.code }}">edit</span> 또는 <span style="{{ site.code }}">e</span>을 사용하면<br>
해당 커밋을 수정할 수 있습니다.<br>


불필요하게 포함된 커밋 내용을 제외하겠습니다.<br>
제외하기 전에 해당 파일들을 백업 한 후 진행하겠습니다.
```
$ cp arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile ~/Makefile
$ cp arch/arm64/boot/dts/rockchip/overlays/odroidm1/pcf8563.dts ~/pcf8563.dts
$ git checkout HEAD~1 arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
$ git rm arch/arm64/boot/dts/rockchip/overlays/odroidm1/pcf8563.dts
```

<br>
이렇게 하면 불필요하게 커밋 내용에 포함되어있던 파일들이 이전 커밋 기준으로 변경되어<br>
커밋 히스토리에서 깔끔하게 사라집니다.<br>
pcf8563.dts 파일은 이전 커밋에서 존재하지 않기 때문에 삭제했습니다.<br>

변경된 내용을 반영합니다.
```
$ git commit --amend
```
```
ODROID-M1: defconfig: pcf8563 is added.

Signed-off-by: Luke go <sangch.go@gmail.com>
Signed-off-by: Steve Jeong <how2soft@gmail.com>
Change-Id: Ie4757602bcbdf9988cd171e984b38fd3853d8499
```

<br>
이제 수정했던 파일들을 새로운 커밋에 올릴 것입니다.
```
$ mv ~/Makefile arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
$ mv ~/pcf8563.dts arch/arm64/boot/dts/rockchip/overlays/odroidm1/pcf8563.dts
$ git add arch/arm64/boot/dts/rockchip/overlays/odroidm1/*
$ git commit -s
```
```
ODROID-M1: dtb/dtbo: pcf8563 is added.

Signed-off-by: Steve Jeong <how2soft@gmail.com>
```
```
$ git rebase --continue
```

rebase가 완료되면 다시 rebase interactive 모드로 진입합니다.
```
...

pick 9e1b1581567d ODROID-M1: net/wireless: update rtl88xx driver, now supports 8811, 8812 and 8814
pick dcfece3adfd9 ODROID-M1: defconfig
s 871d1163ccf9 ODROID-M1: defconfig: pcf8563 is added.
pick 5d028973cb9d ODROID-M1: dtb/dtbo: pcf8563 is added.

...
```

<br>
원본 브렌치와의 diff는 달라지지 않았지만,<br>
이렇게 커밋을 나누고 합치면 다음 업그레이드 때 편해집니다.<br>
현재 존재하는 차이 ~~잘못 올라간 can0 커밋..~~ 는 마지막에 다시 수정할 것입니다.<br>

defconfig관련 커밋을 나눠서 합치는 과정이었습니다.<br>
다시 로그를 확인해 보면,
```
$ git log --oneline arch/arm64/configs/odroidm1_defconfig
3da6a2a96339 ODROID: ov5647: Support ov5647, default setting.
2d6b303557d0 ODROID-M1: defconfig
bedca205a0e1 ODROID-M1: arm64/configs: Supported IMX219.
8563e9a08fc2 ODROID-M1: arm64/dts: add fb_hktft32 and ads7846 dtbo for Hardkernel 3.2 inch TFT
LCD
7506bad7e973 ODROID-M1: staging/fbtft: add fb_hktft32 module for Hardkernel 3.2 inch TFT LCD
dffdb03eb14f ODROID-M1: arch/arm64: add new board Hardkernel's ODROID-M1
```

<br>
결과가 반영되어 있습니다.<br>
이렇게 계속 나누고 합치고 나누고 합치고 반복합니다.
```
$ git log --oneline arch/arm64/configs/odroidm1_defconfig
kee34037298f ODROID-M1: arch/arm64: add new board Hardkernel's ODROID-M1
```

<br>
이렇게 만들어 주면 됩니다.<br>
dtbo도 마찬가지로 같은 과정을 반복해서 모아줍니다.<br>

### 결과

제가 정리한 rebase 결과물 입니다.
```
pick 513d1c0a5607 ODROID-M1: arch/arm64: add new board Hardkernel's ODROID-M1
pick a389d36cdd44 ODROID-M1: dts/dtbo: Introduce device tree overlay
pick 4a1d64fd862a ODROID-M1: dtb/dtbo: Add basic alt functions
pick 001ccc656f3a ODROID-M1: driver/gpiomem: Add gpiomem driver for rk3568
pick 66f75c8812b6 ODROID-M1: driver/gpiomem: Allow access pwm
pick e961036f1468 ODROID-M1: driver/i2c: Add driver "speed" attribution
pick 982f557ca70a ODROID-M1: dtb/dtbo: add fb_hktft32 and ads7846 dtbo for Hardkernel 3.2 inch TFT LCD
pick 7df731c36c65 ODROID-M1: dtb/dtbo: Add NPU device tree overlay
pick 12bd390f70c6 ODROID-M1: dtb/dtbo: Add onewire
pick 7ede7e9ded0e ODROID-M1: dtb/dtbo: add support 115200bps at ttyFIQ0
pick d9dc9dc5a277 ODROID-M1: dtb/dtbo: Add dht11 humidity sensor
pick 0261326519a9 ODROID-M1: dtb/dtbo: Add MODULE_LICENSE to solve can drivers error.
pick a8a2e80f39de ODROID-M1: dtb/dtbo: Add mcp2515 (can module)
pick 4faa3019bf6b ODROID-M1: dtb/dtbo: add to off the blue LED by default
pick e86d0054e3ab ODROID-M1: dtb/dtbo: Add can0 overlay
pick 63008643e7d7 ODROID-M1: dtb/dtbo: pcf8563 is added.
pick 93d03334a447 ODROID-COMMON: phy/realtek: add Wake-on-Lan to Realtek PHY
pick d2638c8cfaec ODROID-M1: arm64/dts: add support Wake-On-Lan
pick a640ab5db33e ODROID-COMMON: drm/panel: ilitek-ili9881c: prepare for adding support for extra panels
pick d5db7d0174ef ODROID-COMMON: drm/panel: ilitek-ili9881c: add support for Feixin K101-IM2BYL02 panel
pick 6694c5cd9d30 ODROID-COMMON: drm/panel: ilitek-ili9881c: add to set dsi format from device tree
pick d830f5638c02 ODROID-COMMON: drm/panel: ilitek-ili9881c: add support for Elida HJ080BE31IA1 panel
pick d8da7bebb809 ODROID-M1: arm64/dts: add reserved memory for PCIe
pick c454f36da77f ODROID-M1: mmc/host: add to hardware reset capability
pick 441ae9d013c5 ODROID-M1: add 'enable-active-high' to PCIe 3.3V regulator
pick 48f67c173d4a ODROID-M1: arch/arm64: add hardware reset property to eMMC
pick 7195cc8e79f8 ODROID-M1: arm64/dts: fix to access SPI flash memory
pick c762c1570568 ODROID-M1: rkflash: enforce to disable 4bit bus access
pick 804937d2788a ODROID-M1: arm64/dts: set 'GPIO0_B0' as 'Headphone Detect'
pick d9c523804eae ODROID-M1: board: add provide board specific information
pick d65af0023d09 ODROID-M1: dtb/dtbo: add 800x1280 8inch touch LCD
pick 446fe109fde1 ODROID-M1: staging/fbtft: add fb_hktft32 module for Hardkernel 3.2 inch TFT LCD
pick 59185e7b8b9e ODROID-M1: dtb/dtbo: add disable-vop2-fixup for ODROID-VU8M
pick 856a51c58171 ODROID-M1: arm64/dts: change PIN_7 as GPIO0_B6
pick 50c6f6472a44 ODROID-M1: board: add a missing offset when reading UUID from sysfs
pick 9c31b9eb659d ODROID: arm64/dts: remotectl: add remotectl to dts
pick 834089665359 ODROID-M1: arm64/dts: switch PWM7 to generic pwm port not IR port
pick 88bf6af2e755 ODROID-COMMON: input/touchscreen: Add Vu5/Vu7+ multitouch driver
pick d7447fec4fe7 ODROID-M1: arm64/dts: Support sound_card for odroid.
pick 03067c615d9e ODROID-M1: arm64/dts: clean up audio properties
pick 318728521d9a ODROID-M1: arm64/dts: add sound card names for HDMI and EARJACK
pick d2a63b739179 ODROID-M1: arm64/dts: rename device tree name to 'display_vu8m.dtbo'
pick faeaedc21357 ODROID-COMMON: net/wireless: add vendor Realtek USB wifi driver
pick e375a562f188 ODROID-COMMON: net/wireless: add vendor Realtek USB wifi driver
pick 1219c625c4cc ODROID-M1: arm64/dts: remove a property '/chosen/disable-vop2-fixup'
pick 4e9e34678be5 ODROID-M1: net/wireless: update rtl88xx driver, now supports 8811, 8812 and 8814
pick ebcc726a4378 ODROID-M1: dtb/dtbo: Support IMX219.
pick 2037b07f0e25 ODROID: imx219: Add v4l2 enum_frame_size function.
pick 39850a206f55 ODROID-M1: driver/imx219: improve driver
pick 029113a87b3f ODROID: imx219: Code refactoring for test pattern.
pick 7b939a472702 ODROID: imx219: Code refactoring.
pick 40caca8a46fa ODROID: imx219: Optimized crop appling.
pick 7b40728d1d44 ODROID: imx477: Add driver.
pick 4c282e639c84 ODROID: imx477: dts/overlay: Add device tree overlay.
pick 24776ab97758 ODROID: imx477: Add rk features, support some video ops.
pick d976c86d17c7 ODROID: ov5647: Support ov5647, default setting.
pick ccbe28f944ad ODROID: ov5647: Add rockchip implementation to support android.
pick 18511af0a79c ODROID: dts/dtbo: Add OV5647 overlays.
pick fda33b6c15a8 ODROID: ov5647: change power gpio setting.
pick 32cf7cae854c ODROID-M1: ov5647: Support flip feature.
pick 6bddcbda5862 ODROID-M1: ov5647: Support test pattern feature.
pick 44f2fa4fac4f ODROID-M1: ov5647: changed color format x8 to x10.
pick 7f599b243174 ODROID-M1: ov5647: Support v4l2 features.
pick 45d644573f0c ODROID-M1: ov5647: Code Refactory.
```

<br>
conflict를 풀어주느라 애를 먹었네요..<br>

원본 브렌치와 비교 해보면
```
$ git diff odroidm1-4.19.y
```
```
```

<br>
이렇게 차이가 없으면 성공입니다.<br>
<br>

+<br>
제 실수를 어떻게 커버했냐면
```
$ tig arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
```

<br>
<p align="center">
  <img src="/assets/gif/legacy/m1-tig.gif" alt="m1-tig" width="640" height="480"><br>
  <span style="{{ site.img }}">Git log</span>
</p>
<br>

<br>
그냥 ~~노가다~~ 열심히 했습니다.<br>

이제 remote 저장소에 푸쉬하고 테스트가 끝나면, 저와 협업하는 분들에게 <span style="{{ site.code }}">bundle</span> 로 커밋들을 공유할 예정입니다.<br>
