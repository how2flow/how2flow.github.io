---
permalink: /legacy/yolov8s-with-npu
title: "yolov8 모델 사용하기 (CPU vs NPU)"
excerpt: "오드로이드 M1S의 NPU를 사용해서 Yolo 모델을 돌려보고 오브젝트 디텍팅도 해보겠습니다."
header:
  teaser: /assets/images/legacy/rocket.png
categories:
  - SBC
tags:
  - ai
  - cpu
  - devfreq
  - governor
  - ldconfig
  - m1s
  - npu
  - odroid
  - rknn
  - rknn_model_zoo
  - ultralytics
  - yolo
  - yolov5
  - yolov8

toc: true
---

## 소개

NPU는 예전에 한번 포스팅 한 적이 있습니다.<br>
요즘 핫한 AI와 관련된 키워드 인데요, 오드로이드 M1으로 사용해 본적이 있습니다.<br>
M1으로 NPU를 사용했을 땐 <span style="{{ site.code }}">mobilenet</span> 과 <span style="{{ site.code }}">yolov5s</span> 를 사용했었습니다.<br>
간단한 NPU 설명이 포함된 [이전 포스팅](/posts/npu/).<br>

이번에는 오드로이드 M1S를 사용해서 <span style="{{ site.code }}">yolov8s</span> 모델을 사용해 볼 예정입니다.<br>

"YOLO"는 "You Only Look Once"의 약자로, 객체 감지(Object Detection)를 위한 심층 학습 알고리즘 중 하나입니다.<br>
이 알고리즘은 이미지나 비디오에서 여러 객체의 위치를 식별하고 분류할 수 있습니다.<br>

### 진행 목표

이번에는 특별히 NPU의 강력함을 더 와닿게 하기 위해서,<br>
CPU로 yolov8s를 사용하는 속도와 비교해 보겠습니다.<br>

## yolov8 사용하기 (CPU)

yolov8이 업데이트 되면서 훨씬 간단하게 Yolo 모델들을 사용할 수 있게 되었습니다.<br>
<span style="{{ site.code }}">CLI</span> 에서 바로 사용 가능하게끔 지원됩니다.<br>

[관련 문서](https://docs.ultralytics.com/usage/cli/)를 제공합니다.<br>

### CPU governor 세팅하기

최대 성능을 뽑아내기 위해서 <span style="{{ site.code }}">governor</span> 를 <span style="{{ site.code }}">performance</span> 로 세팅합니다.
```
$ echo performance | sudo tee /sys/devices/cpu/system/cpu0/cpufreq/scaling_governor
```
<br>

yolo 명령어로 오브젝트 디텍팅 할 때 <span style="{{ site.code }}">top</span> 명령어를 통해<br>
싱글코어로 돌아가는 것을 확인했습니다. <span style="{{ site.code }}">cpu0</span> 만 세팅하면 됩니다.<br>

멀티 프로세싱이나 GPU를 사용하기 위해서는 CUDA 디바이스를 따로 물려야 하는데,<br>
주변장치 없이 보드의 기본 세팅으로 비교하기 위해 그래도 진행합니다.<br>

### python 패키지 설치

yolov8 사용을 위한 패키지를 설치합니다.
가상환경에서 실행 가능합니다.
```
$ sudo apt install python3-dev python3-pip python3-venv
$ sudo apt install libgl1-mesa-glx
$ python3 -m venv .venv/yolov8
$ source .venv/yolov8/bin/activate
(yolov8) $ python3 -m pip install --upgrade pip
(yolov8) $ python3 -m pip install ultralytics opencv-python onnx onnxruntime
```
<br>

### onnx 모델 사용하기

테스트는 onnx모델로 진행할 것입니다.<br>
과정은 pt 모델을 onnx모델로 추출한 다음,<br>
onnx모델로 inference 합니다.
```
(yolov8) $ yolo export model=yolov8s.pt format=onnx
```

yolo detect 명령어 입니다.<br>
이미지를 설정하지 않으면 기본으로 <span style="{{ site.code }}">bus.jpg</span> <span style="{{ site.code }}">jidan.jpg</span> 두가지를 사용합니다.<br>
이미지는 <span style="{{ site.code }}">source={input_image}</span> 으로 설정할 수 있습니다.
```
$ yolo detect predict model=yolov8s.onnx imgsz=640 conf=0.25
```
<span style="{{ site.code }}">model</span> 은 <span style="{{ site.code }}">yolov8s.onnx</span> 이미지 크기는 640 전용입니다.<br>
<span style="{{ site.code }}">confidence</span> 는 0.25로 세팅합니다.<br>

### 결과 확인

결과는 다음과 같이 나타납니다.
```
...
image 1/2 /home/odroid/.venv/yolov8/lib/python3.10/site-packages/ultralytics/assets/bus.jpg: 640x640 4 persons, 1 bus, 2338.5ms
image 2/2 /home/odroid/.venv/yolov8/lib/python3.10/site-packages/ultralytics/assets/zidane.jpg: 640x640 2 persons, 1 tie, 2154.3ms
...
```

<span style="{{ site.code }}">bus.jpg</span> 로 비교할 것이기 때문에 첫번째 결과 <span style="{{ site.code }}">2338.5ms</span> 를 사용할 것입니다.<br>
output은 <span style="{{ site.code }}">run/detect/predict<span> 에 저장됩니다.<br>

<p align="center">
  <img src="/assets/images/legacy/input-cpu-bus.jpg" alt="input-cpu-bus" width="320" height="240">
  <img src="/assets/images/legacy/output-cpu-bus.jpg" alt="output-cpu-bus" width="320" height="240"><br>
  <span style="{{ site.img }}">[picture 1] Result of yolov8 demo (cpu)</span>
</p>
<br>

## yolov8 사용하기 (NPU)

### NPU governor 세팅하기

속도를 최대한으로 끌어올리기 위해 governor를 performance로 세팅하겠습니다.
```
$ echo performance | sudo tee /sys/class/devfreq/fde40000.npu/governor
```
<br>

### npu 오버레이 추가하기

M1S는 전작인 M1과 다르게, <span style="{{ site.code }}">petitboot</span> 가 없습니다.<br>
따라서 <span style="{{ site.code }}">petitboot</span> 를 스킵하는 과정은 없습니다.<br>

rknpu 오버레이를 추가합니다.
```
$ sudo vi /boot/config.ini
```
```
[generic]
overlay_resize=16384
overlay_profile=
overlays="rknpu"

[overlay_custom]
overlays="i2c0 i2c1"

[overlay_hktft32]
overlays="hktft32 ads7846"
```

리부팅 합니다.<br>
리부팅 이후 커널 모듈 <span style="{{ site.code }}">insmod</span>
```
$ sudo modprobe rknpu
$ lsmod | grep rknpu
rknpu                  49152  0
```
<br>

### 의존성 확인하기

M1S에서 제공하는 NPU에는 크게 두가지 의존성이 있습니다.<br>
<span style="{{ site.code }}">rkrga</span> 드라이버 버전과 <span style="{{ site.code }}">librga</span> 버전입니다.<br>
전자는 rga 드라이버 자체의 버전이고, 후자는 예제에서 사용할 rga 관련 라이브러리 버전입니다.<br>

버전 확인은 [공식 문서](https://github.com/airockchip/librga/blob/main/docs/Rockchip_Developer_Guide_RGA_EN.md)에서 확인할 수 있습니다.<br>
<br>
<p align="center">
  <img src="/assets/images/legacy/m1-corr.ver.png" alt="m1-corr.ver" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 2] correspondence between versions</span>
</p>
<br>

보드에서 <span style="{{ site.code }}">rkrga</span> 드라이버 버전을 확인해 보겠습니다.
```
$ sudo su
# cat /sys/kernel/debug/rkrga/driver_version
RGA2 Device Driver: v2.1.0
# exit
```

<br>
현재 드라이버 버전은 v2.1.0입니다.<br>
librga 버전을 1.0.0 ~ 1.3.2 버전을 사용하거나 1.9.0 버전 이상을 사용하면 될 것 같습니다.<br>

### 예제 다운로드

예제 프로젝트를 다운로드 받고, 먼저 librga 버전을 확인합니다.
```
$ cd
$ git clone -b bench https://github.com/how2flow/rknn_model_zoo.git
$ cd rknn_model_zoo/3rdparty/librga/Linux/aarch64
$ strings librga.so | grep rga_api | grep version
rga_api version 1.9.1_[4]
```
librga가 rkrga 드라이버와 호환이 되는 버전이니 계속 진행합니다.<br>

<br>
빌드스크립트를 실행합니다.
```
$ cd ~/rknn_model_zoo
$ sudo chmod +x build-linux.sh
$ ./build-linux.sh -a aarch64 -t rk3566 -d yolov8
```
<br>

예제가 빌드되었으면 결과를 확인합니다.
```
$ cd install/rk356x_linux_aarch64/rknn_yolov8_demo
$ sudo ./rknn_yolov8_demo model/rk356x_yolov8s.rknn model/bus.jpg
```
<br>

<details>
  <summary><span style="{{ site.code }}">error while loading shared libraries...</span></summary>
  <p>
  <br>
  만약 다음과 같은 에러가 나타난다면 shared object가 링크되지 않아서 발생하는 문제입니다.
  <pre><code>
  ./rknn_yolov8_demo: error while loading shared libraries: librknnrt.so: cannot open shared object file: No such file or directory
  </code></pre>
  
  해당 .so 파일은 rknn_yolov8_demo 실행파일과 동일한 경로에 존재하는<br>
  lib 디렉토리 안에 있습니다. 링크 시켜줍니다.
  <pre><code>
  $ export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:./lib
  $ sudo ldconfig
  </code></pre>
  <br>
  
  다시 실행합니다.

  <pre><code>
  $ sudo ./rknn_yolov8_demo model/rk356x_yolov8s.rknn model/bus.jpg
  </code></pre>

  </p>
</details>

### 결과 확인

<br><br>
```
  ...
  rknn_run
  loop count = 10 average run 74.817000 ms
  write_image path: out.png width=640 height=640 channel=3 data=0x55a957e030
```
위와 같은 로그가 나타나면 성공입니다.<br>
여러번 실행해 보았는데 약 <span style="{{ site.code }}">73 ~ 75 ms</span> 왔다갔다 합니다.<br>
<br>
<p align="center">
  <img src="/assets/images/legacy/input-bus.jpg" alt="input-bus" width="320" height="240">
  <img src="/assets/images/legacy/output-bus.jpg" alt="output-bus" width="320" height="240"><br>
  <span style="{{ site.img }}"> [picture 3] Result of yolov8 demo (npu)</span>
</p>
<br>

사진의 사이즈가 약간 다른것은 모델 학습시 사용한 이미지의 차이가 있기 때문입니다.<br>
하지만 결과에 큰 영향을 줄 만큼은 아닙니다.<br>

## CPU vs NPU

inference time을 측정한 결과입니다.<br>
약 100회 정도 실행한 평균값이고 소수점 셋째자리에서 반올림 했습니다.<br>
<br>

| yolov8 | CPU | NPU |
|:-------|:---:|:---:|
| M1S    | 2338.84 | 74.20 |

<br>

단위는 <span style="{{ site.code }}">ms</span> 이고 <span style="{{ site.code }}">confidence</span> 는 동일하게 <span style="{{ site.code }}">0.25</span> 입니다.<br>
속도는 NPU가 CPU 대비 약 32배 정도 빠릅니다<br>
