---
categories:
- linux
excerpt: A step-by-step guide on upgrading the ODROID-M1 kernel from 4.19.y to 5.10.y
  using Git Rebase.
header:
  teaser: /assets/images/posts/thumbnails/note.jpg
layout: single
tags:
- kernel
- rebase
- git
- odroid
- embedded
title: '[Linux] Kernel Upgrade Guide with Git Rebase (Part 1)'
toc: true
redirect_from:
- /documents/linux/upgrade-kernel/
- /legacy/upgrade-kernel/
- /documents/linux/upgrade-kernel
- /legacy/upgrade-kernel
---
This post documents the process of upgrading the ODROID-M1 kernel version. As of the writing of this post, ODROID-M1 supports Linux kernel version 4.19 (<span style="{{ site.code }}">linux-image-4.19.219-odroid-arm64</span>).

With the latest BSP update, we are upgrading the kernel to a newer version. Since the official release is not yet out, only parts of the process are shared here.

### Prerequisites
- PC (Ubuntu 20.04 LTS or later)
- ODROID-M1 board
- SSH-enabled environment

## Preparing Source Code and Commits

When upgrading the Linux kernel, especially for BSPs, you must migrate all existing functional commits to the new version to ensure continued feature support. 

Managing a large number of commits across collaborative teams requires careful organization. It is more efficient for the original author of a commit to handle its migration and testing. Therefore, organizing commits before the migration is a crucial first step.

### Downloading Source Code
Download the kernel source using the following command:
```bash
$ git clone -b odroidm1-4.19.y --single-branch https://github.com/hardkernel/linux
```

### Backing Up Original Logs
Before rearranging commits, back up the original log to easily compare changes later.
```bash
$ cd linux
$ git log --oneline > ~/odroidm1-4.19_original_log
```

### Creating a Work Branch
Create a dedicated branch for organizing commits.
```bash
$ git checkout -b odroidm1-5.10.y-dev
```
The overall workflow consists of repeating **Rebase -> Diff** until the work branch matches the source state of the original branch.

---

## Mastering Git Rebase

### Before Starting Rebase
Use `git rebase` to organize commits. You specify the commit immediately preceding the range you want to edit. We use the `-i` (`--interactive`) option to handle large volumes of commits.

```bash
$ git rebase -i <base-commit-hash>
```
The interactive editor will list commits from oldest (top) to newest (bottom).

#### Best Practices for Rebase
1. **Maintain Order:** Try to keep the original chronological order of commits.
2. **Minimize Movement:** Move the smallest number of commits possible.
3. **Stay Close:** Keep moved commits as close to their original relative positions as possible.

### Relocating Non-Assigned Commits (e.g., Camera)
I was responsible for **I/O Peripherals** and **NPU**. To streamline handover, I moved other components (like Camera drivers: `ov5647`, `imx219`, `imx477`) to the top of the history.

While there is no single right way to arrange commits, the order you choose significantly impacts the difficulty of resolving conflicts.

### Handling Conflicts
During rebase, you will likely encounter conflicts, especially when parent commits change.
```text
CONFLICT (content): Merge conflict in arch/arm64/boot/dts/rockchip/overlays/odroidm1/Makefile
error: could not apply ...
```
When a conflict occurs:
1. Identify the conflict: `git diff`
2. Resolve the manual changes in the files.
3. Mark as resolved: `git add <file>`
4. Continue the rebase: `git rebase --continue`

### Using `edit` to Split Commits
If a single commit contains unrelated changes (e.g., both `defconfig` updates and a new `dts` file), use the `edit` (or `e`) command in interactive mode to split it.

1. Mark the commit as `edit` in the rebase list.
2. When Git stops at that commit, reset the unnecessary files: `git checkout HEAD~1 <file>`
3. Amend the commit: `git commit --amend`
4. Add the reset files as a new commit: `git add <file> && git commit -m "New clean commit"`
5. Continue: `git rebase --continue`

## Result
After numerous iterations of `rebase -i`, `squash`, and `edit`, the final history becomes clean and modular, making it much easier to `cherry-pick` into the new kernel version.

To verify success, ensure that a `diff` between your organized branch and the original branch shows no differences:
```bash
$ git diff odroidm1-4.19.y
```
If the output is empty, the rebase was successful without losing any code integrity.
