---
permalink: /legacy/make_initramfs
title: "[Linux] Initramfs" 
excerpt: "Initramfs 만드는 방법과 수정하는 방법"
header:
  teaser: /assets/images/legacy/note.jpg
categories:
  - Linux
tags:
  - boot
  - boot.cmd
  - boot.scr
  - cpio
  - initramfs
  - initramfs64
  - mkimage
toc: true
---

## 소개

ARMv8 아키텍처의 CPU 칩에 리눅스 OS를 올리고 부팅하기 위해서는 최소 3가지가 충족되어야 합니다.<br>
리눅스 커널을 부팅할 부트로더, 리눅스 커널, 그리고 그 위에 올라가는 리눅스 파일 시스템이 필요합니다.<br>
부트로더는 U-BOOT를 많이 사용합니다.<br>
리눅스 커널이 올라가고 파일시스템이 올라가기 전에 기본적인 디바이스를 인식하고 마운트 하는 역할을<br>
하는 것이 initramfs 입니다.<br>
ram에 올라가는 파일 시스템으로, 기본적인 파일 시스템 구성을 마치면 pivot_root 이후에 우리가 많이 사용하는<br>
우분투 파일시스템 등이 올라가게 됩니다.<br>

해당 페이지에서 initramfs를 사용하지 않고 부팅하는 방법이 소개되어 있습니다.<br>
[initramfs 없이 리눅스 부팅하기](/documents/linux/boot-linux-noinitrd/)<br>
일반적인 방법이 아니며, initramfs가 존재하기 이전의 방식입니다.<br>

위의 방법과 다르게 파일 시스템 트리를 만들고, init script를 추가한 다음,<br>
mkimage 명령으로 파일 시스템을 만들어 주면 됩니다.<br>
CONFIG는 CONFIG_BLK_DEV_INITRD를 built-in 시켜 줘야 합니다.
```
CONFIG_BLK_DEV_INITRD=y
```
<br>

커널 설정을 해주고 난 다음, 사용할 initramfs를 만들면 됩니다.<br>

## mkimage

mkimage는 u-boot-tools 패키지에 포함되어 있습니다.



### initramfs 만들기

initramfs 파일은 mkimage 으로 만들 수 있습니다.
```
$ mkimage -A ${ARCH} -O linux -T ramdisk -d {datafile} uInitrd.img
```
<br>

기본적인 파일트리는 기존의 클라우드 이미지들을 참고하면 좋습니다.<br>
파일트리를 만들었다면 다음 과정을 통해 initramfs 이미지를 만들 수 있습니다.
```
$ croot # move to file-tree root path
$ find . | cpio -H newc -ov --owner root:root > ../initramfs.cpio
$ cd ..
$ gzip initramfs.cpio # No compression format determined
$ mkimage -A arm64 -O linux -T ramdisk -d initramfs.cpio.gz uInitrd.img
```
<br>

lzop 형식으로 다음과 같이 압축할 수 있습니다.
```
$ croot # move to file-tree root path
$ find . | cpio -H newc -ov --owner root:root > ../initramfs.cpio
$ cd ..
$ lzop -9 initramfs.cpio
```
<br>

파일 트리에는 init 스크립트가 포함되어 있어야 합니다.<br>
U-boot의 root 파라미터는 init 스크립트의 위치를 지정합니다.<br>

### 이미 존재하는 아카이브 initramfs 에서 필요한 파일 추가하기

기존의 initramfs파일에서 특정 파일만 추가하여 다시 아카이브 하는 방법 입니다.
```
$ lzop -d initramfs.cpio.lzo
$ mkdir -p initramfs/home/root # add files that you want in 'home/root'
$ cd initramfs
$ find . -depth | cpio -oA -H newc -F ../initramfs.cpio
```
<br>

#### boot script 만들기
추가로 부트 스크립트를 따로 만드는 방법은 다음과 같습니다.<br>
U-boot에서 입력해야할 커맨드들을 boot.cmd 파일에 저장한 이후 다음을 실행합니다.
```
$ mkimage -A arm64 -O linux -T script -C none -d boot.cmd boot.scr
```
<br>
