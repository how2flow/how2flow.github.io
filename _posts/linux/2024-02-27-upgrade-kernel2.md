---
categories:
- linux
excerpt: Upgrading (rebasing) the kernel version for ODROID-M1.
header:
  teaser: /assets/images/posts/thumbnails/note.jpg
layout: single
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
title: '[Linux] Kernel Version Upgrade 2 (with ODROID-M1)'
toc: true
redirect_from:
- /documents/linux/upgrade-kernel2/
- /legacy/upgrade-kernel2/
- /documents/linux/upgrade-kernel2
- /legacy/upgrade-kernel2
---
September seems to be the month for part 2s.<br>
My posts keep getting delayed because I have a lot going on,<br>
but I'll try to work harder.<br>

## Prerequisites

Prepare the following:
```
- PC (Ubuntu 20.04LTS or later)
- ODROID-M1 board (x1)
- SSH-enabled environment
```

## Kernel Upgrade

### Commit History

In the [previous post](/posts/upgrade-kernel/), we went through the process of organizing commits before the upgrade.<br>
Now, we need to actually apply those commits to the BSP.<br>
Before doing that, let's review the history of the BSP and the M1.<br>

First, the board has its main chips.<br>
The ODROID-M1 is equipped with the RK3568.<br>
The RK3568 includes a CPU, GPU, NPU, etc., and is provided by Rockchip.<br>

To make this chip operational, the manufacturer (Rockchip) provides software, including the kernel, along with an EVB (Evaluation Board) that connects various I/Os for testing.<br>
This software package is called a BSP (Board Support Package).<br>

Since I will be upgrading the kernel, I'll only look at the kernel history.<br>
Rockchip's BSP tends to follow the Linux kernel mainline.<br>

The [linux mainline](https://github.com/torvalds/linux) looks like this:
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

It's a structure where commits accumulate in a straight line by version.<br>

Rockchip's BSP (RK3568) looks like this:
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

The kernel update for the M1 is based on the BSP (as of 2023.09.01, the M1 BSP is `rk356x_linux_20221220`).
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

The BSP kernel is either built on top of previously provided commits, like this:
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

Or, there are cases where the kernel version itself is upgraded:
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

Looking at it again alongside the M1's history:
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
It looks like the diagram above.<br>

<br>
We can see the continuous updates on the M1's 4.19 kernel and the `odroidm1-5.10.y-dev` branch created in the previous post to organize the commits.<br>
Now, we need to place these organized commits on top of the 5.10 version BSP.<br>

In summary,<br>
all commits between the <span style="{{ site.code }}">rk356x_linux_20221220</span> branch and the <span style="{{ site.code }}">odroidm1-5.10.y-dev</span> branch must be moved<br>
on top of the <span style="{{ site.code }}">rk356x_linux_bbbbbbbb</span> branch.<br>

There are two main ways to do this: <span style="{{ site.code }}">cherry-pick</span> and <span style="{{ site.code }}">rebase</span>.<br>
I mostly use <span style="{{ site.code }}">cherry-pick</span> when performing an upgrade.<br>

This is because the number of commits is not overwhelmingly large (around 40), and I apply and test them one by one.<br>
It also makes resolving conflicts much easier.<br>

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
The cherry-pick method is straightforward.<br>
Every M1 commit that was updated on top of the <span style="{{ site.code }}">rk356x_linux_20221220</span> branch<br>
is simply cherry-picked <U>sequentially one by one</U> onto the new BSP commits.<br>

After <span style="{{ site.code }}">cherry-pick</span>ing all the commits and deleting the old branch, the result looks like this:
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
This state represents a completed upgrade.<br><br>

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
You need to <span style="{{ site.code }}">rebase</span> all commits between the <span style="{{ site.code }}">rk356x_linux_20221220</span> branch and the <span style="{{ site.code }}">odroidm1-5.10.y-dev</span> branch.<br>

For <span style="{{ site.code }}">rebase</span>, you use the <span style="{{ site.code }}">--onto</span> option.<br>

If you just run a standard rebase, for example (while at <span style="{{ site.code }}">HEAD: odroidm1-5.10.y-dev</span>):
```
$ git rebase rk356x_linux_bbbbbbbb
```

This will attempt to apply all commits with a different history from the target branch (<span style="{{ site.code }}">rk356x_linux_bbbbbbbb</span>).<br>

Because this would also rebase all the commits down to right above the Linux mainline 4.19 version, you must use the <span style="{{ site.code }}">--onto</span> option.<br>

<br>
Using the <span style="{{ site.code }}">--onto</span> option prevents rebasing every commit with a different history than the target branch.<br>
Instead, it specifies an exact range.<br>

<span style="{{ site.code }}">git rebase --onto 'target' 'a' 'b'</span><br>
This rebases commits from 'a' to 'b' onto 'target'.<br>

The command for the M1 upgrade is as follows (while at <span style="{{ site.code }}">HEAD: odroidm1-5.10.y-dev</span>):
```
$ git rebase --onto rk356x_linux_bbbbbbbb rk356x_linux_20221220 odroidm1-5.10.y-dev
```

Alternatively,<br>
if you know the exact number of commits to move (say, 10), you can do this (while at <span style="{{ site.code }}">HEAD: odroidm1-5.10.y-dev</span>):
```
$ git rebase --onto rk356x_linux_bbbbbbbb HEAD~10 odroidm1-5.10.y-dev
```
You can rebase this way as well.<br>
However, I do not generally recommend the rebase approach itself.<br>

Of course, you have to resolve any conflicts that arise during the rebase process.<br>
Once the rebase is complete, it takes the following shape:
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

## Release

Testing is ongoing. Look forward to the next upgraded M1 release!<br>
