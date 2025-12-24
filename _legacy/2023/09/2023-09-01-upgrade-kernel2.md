---
permalink: /legacy/upgrade-kernel2/
title: "[Linux] 커널 버전 업그레이드 2(with ODROID-M1)"
excerpt: "ODROID -M1의 커널 버전 올리기 (rebase)"
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

9월달은 2편의 날입니다.<br>
이것 저것 하는게 많아서 포스팅이 계속 늦어지는데,<br>
더 분발해야 겠습니다.<br>

## 준비물

다음을 준비합니다.
```
- PC (Ubuntu 20.04LTS or later)
- ODROID-M1 board (x1)
- ssh 사용이 가능한 환경
```

## 커널 업그레이드

### 커밋 히스토리

[이전 포스팅](/posts/upgrade-kernel/) 에서는 업그레이드 하기 전에 커밋을 정리하는 과정이었습니다.<br>
이제 실제 bsp에 커밋을 올리는 과정을 해야 합니다.<br>
그전에 bsp와 M1의 히스토리를 리뷰하겠습니다.<br>

먼저 보드에는 메인이 되는 칩들이 있습니다.<br>
ODROID-M1은 RK3568이 탑재된 모델입니다.<br>
RK3568은 CPU, GPU, NPU 등이 포함되고 있으며 Rockchip에서 제공하는 칩입니다.<br>

이 칩을 동작시킬 수 있게 커널을 포함한 소프트웨어들과 테스트를 할 수 있게 여러 I/O가 연결된 evb를<br>
제조사(Rockchip)에서 제공하는데 이걸 bsp(board support package)라고 합니다.<br>

저는 커널을 업그레이드 할 것이기 때문에 커널 히스토리만 보겠습니다.<br>
Rockchip의 bsp는 linux kernel mainline을 따라가는 특징이 있습니다.<br>

[linux mainline](https://github.com/torvalds/linux)은,
```
...
 *
 *
...
 * --- 5.10.y
...
 *
 * --- 4.19.y
```
<br>

한줄로 쭉 커밋이 버전별로 쌓여가는 형태입니다.<br>

Rockchip의 bsp(RK3568)는 다음과 같습니다.
```
...
 *
 * --- 5.10.y
 *
 *
 *
 *
 *     * --- rk356x_linux_20221220
 *    /
 *  ...
 *  /
 * *
 */
 * --- 4.19.y

```

M1의 커널 업데이트는 bsp 기준(2023.09.01 기준 M1 bsp:rk356x_linux_20221220) 으로 진행됩니다.
```
       * --- odroidm1-4.19.y
       *
       *
      ...
       *
       * --- rk356x_linux_20221220
      /
    ...
    /
   *
  /
 * --- 4.19.y

```
<br>

bsp 커널은 기존에 제공되었던 커밋에 새로 쌓아 올라가거나
```
       * --- rk356x_linux_aaaaaaaa
       *
       *
      ...
       *
       * --- rk356x_linux_20221220
      /
    ...
    /
   *
  /
 * --- 4.19.y
```
<br>

커널 버전 자체를 업그레이드를 하는 경우가 있습니다.
```
                              <Linux>
                                 * 
rk356x_linux_bbbbbbbb ---  *     *
                            \    *
                            ...  *
                              \  *
                               * *
                                \*
                                 * --- 5.10.y
                                ...
                                 *
                                 *
                                 *
                                 *     * --- rk356x_linux_aaaaaaaa
                                 *     *
                                 *    ...
                                 *     *
                                 *     * --- rk356x_linux_20221220
                                 *    /
                                 *  ...
                                 *  /
                                 * *
                                 */
                                 * --- 4.19.y
```
<br>

M1의 히스토리와 함께 다시 보면
```
                              <Linux>
                                 * 
rk356x_linux_bbbbbbbb ---  *     *
                            \    *
                            ...  *
                              \  *
                               * *
                                \*
                                 * --- 5.10.y
                                ...
                                 *
                                 *
                                 *
                                 *     * --- odroidm1-4.19.y
                                 *     *
                                 *     *   * --- (HEAD) -> odroidm1-5.10.y-dev
                                 *    ... /
                                 *     * *
                                 *     */
                                 *     * --- rk356x_linux_20221220
                                 *    /
                                 *  ...
                                 *  /
                                 * *
                                 */
                                 * --- 4.19.y
```
위의 그림처럼 됩니다.<br>

<br>
계속 업데이트 해 왔던 M1의 4.19 커널과 전 포스팅에서 커밋을 정리하기 위해 만든 odroidm1-5.10.y-dev 브렌치도 보입니다.<br>
이제 정리된 커밋들을 5.10 버전의 bsp 위로 올려야 합니다.<br>

정리하자면,<br>
<span style="{{ site.code }}">rk356x_linux_20221220</span> 브렌치부터 <span style="{{ site.code }}">odroidm1-5.10.y-dev</span> 브렌치 사이에 있는 모든 커밋들을<br>
<span style="{{ site.code }}">rk356x_linux_bbbbbbbb</span> 브렌치 위로 올려야 합니다.<br>

방법은 크게 <span style="{{ site.code }}">cherry-pick</span> , <span style="{{ site.code }}">rebase</span> 2 가지 방법이 있습니다.<br>
저는 업그레이드를 할 때 <span style="{{ site.code }}">cherry-pick</span> 을 주로 사용합니다.<br>

커밋 수도 엄청 많지 않고 (약 40개) 하나씩 올리고 테스트하기 때문입니다.<br>
conflict 해결도 훨씬 쉽습니다.<br>

### cherry-pick

```
                              <Linux>
                                 * 
rk356x_linux_bbbbbbbb ---  *     *
                            \    *
                            ...  *
                              \  *
                               * *
                                \*
                                 * --- 5.10.y
                                ...
                                 *
                                 *
                                 *
                                 *     * --- odroidm1-4.19.y
                                 *     *
                                 *     *   * --- (HEAD) -> odroidm1-5.10.y-dev
                                 *    ... /
                                 *     * *
                                 *     */
                                 *     * --- rk356x_linux_20221220
                                 *    /
                                 *  ...
                                 *  /
                                 * *
                                 */
                                 * --- 4.19.y
```
cherry-pick 방법은 간단합니다.<br>
<span style="{{ site.code }}">rk356x_linux_20221220</span> 브렌치 위로 업데이트 했던 모든 M1 커밋들을<br>
하나씩 새로운 bsp 커밋 위로 <U>순서대로 하나씩</U> cherry-pick 합니다.<br>

모든 커밋들을 <span style="{{ stie.code }}">cherry-pick</span> 하고 기존의 브렌치를 삭제하면 다음과 같은 결과가 나타납니다.
```
                              <Linux>
                                ...
                                 *
                                 *
odroidm1-5.10.y-dev ---    *     *
                           *     *
                          ...    *
                           *     * 
rk356x_linux_bbbbbbbb ---  *     *
                            \    *
                            ...  *
                              \  *
                               * *
                                \*
                                 * --- 5.10.y
                                ...
                                 *
                                 *
                                 *
                                 *     * --- odroidm1-4.19.y
                                 *     *
                                 *     *
                                 *    ...
                                 *     *
                                 *     *
                                 *     * --- rk356x_linux_20221220
                                 *    /
                                 *  ...
                                 *  /
                                 * *
                                 */
                                 * --- 4.19.y
```
위의 형태가 업그레이드 완료된 상태입니다.<br><br>

### rebase

```
                              <Linux>
                                 * 
rk356x_linux_bbbbbbbb ---  *     *
                            \    *
                            ...  *
                              \  *
                               * *
                                \*
                                 * --- 5.10.y
                                ...
                                 *
                                 *
                                 *
                                 *     * --- odroidm1-4.19.y
                                 *     *
                                 *     *   * --- (HEAD) -> odroidm1-5.10.y-dev
                                 *    ... /
                                 *     * *
                                 *     */
                                 *     * --- rk356x_linux_20221220
                                 *    /
                                 *  ...
                                 *  /
                                 * *
                                 */
                                 * --- 4.19.y
```
<span style="{{ site.code }}">rk356x_linux_20221220</span> 브렌치부터 <span style="{{ site.code }}">odroidm1-5.10.y-dev</span> 브렌치 사이에 있는 모든 커밋들을<br>
<span style="{{ site.code }}">rebase</span> 해야 합니다.<br>

<span style="{{ site.code }}">rebase</span>는 <span style="{{ site.code }}">--onto</span> 옵션을 사용하면 됩니다.<br>

그냥 rebase를 바로 하면, 예를 들어서 ( <span style="{{ site.code }}">HEAD: odroidm1-5.10.y-dev</span> )
```
$ git rebase rk356x_linux_bbbbbbbb
```

이렇게 하면, target 브렌치인 <span style="{{ site.code }}">rk356x_linux_bbbbbbbb</span>의 히스토리와 다른 히스토리를 가진 모든 커밋들이 새로 올라가게 됩니다.<br>

linux mainline 4.19 버전 커밋 바로 위의 커밋까지 함께 rebase를 하게 되기 때문에<br>
<span style="{{ site.code }}">--onto</span> 옵션을 사용해야 합니다.<br>

<br>
<span style="{{ site.code }}">--onto</span> 옵션을 사용하면 타겟 브렌치와 히스토리가 다른 모든 커밋을 rebase하는 것이 아니고,<br>
그 중에서 특정 범위를 지정합니다.<br> 

<span style="{{ site.code }}">git rebase --onto 'target' 'a' 'b'</span><br>
a 부터 b 사이 커밋을 target에 rebase 하는 것입니다.<br> 

M1 업그레이드 명령은 다음과 같습니다. ( <span style="{{ site.code }}">HEAD: odroidm1-5.10.y-dev</span> )
```
$ git rebase --onto rk356x_linux_bbbbbbbb rk356x_linux_20221220 odroidm1-5.10.y-dev
```

혹은,<br>
내가 옮길 커밋의 개수를 알고 있고 10개라고 하면, ( <span style="{{ site.code }}">HEAD: odroidm1-5.10.y-dev</span> )
```
$ git rebase --onto rk356x_linux_bbbbbbbb HEAD~10 odroidm1-5.10.y-dev
```
이렇게도 rebase 할 수 있습니다.<br>
다만, rebase 방식 자체를 권장하지 않습니다.<br>

물론 rebase 과정에서 나타나는 모든 conflict를 해결해야 합니다..<br>
rebase가 완료되면 다음과 같은 형태가 됩니다.
```
                              <Linux>
                                ...
                                 *
                                 *
odroidm1-5.10.y-dev ---    *     *
                           *     *
                          ...    *
                           *     * 
rk356x_linux_bbbbbbbb ---  *     *
                            \    *
                            ...  *
                              \  *
                               * *
                                \*
                                 * --- 5.10.y
                                ...
                                 *
                                 *
                                 *
                                 *     * --- odroidm1-4.19.y
                                 *     *
                                 *     *
                                 *    ...
                                 *     *
                                 *     *
                                 *     * --- rk356x_linux_20221220
                                 *    /
                                 *  ...
                                 *  /
                                 * *
                                 */
                                 * --- 4.19.y
```
<br>

## 릴리즈

계속 테스트 중에 있습니다. 업그레이드 되는 다음 M1을 기대해 주세요.!<br>
