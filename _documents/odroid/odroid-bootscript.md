---
permalink: /documents/odroid-bootscript/
title: "Update Bootscript Of Odroid"
excerpt: "how to update bootscript of odroid?"
comments: false
---

## Command

Use mkimage
```
$ sudo mkimage -A arm64 -T script -C none -d boot.cmd boot.scr
```
<br>

<span style="{{ site.code }}">boot.cmd</span> is a bootscript txt file.<br>
<span style="{{ site.code }}">boot.scr</span> is a bootscript of odroids.<br>
