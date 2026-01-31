---
categories:
- odroid
comments: false
excerpt: how to update bootscript of odroid?
header:
  teaser: /assets/images/posts/thumbnails/note.jpg
layout: single
title: '[ODROID] Update Bootscript Of Odroid'
redirect_from:
- /documents/odroid/odroid-bootscript/
- /legacy/odroid-bootscript/
- /documents/odroid/odroid-bootscript
- /legacy/odroid-bootscript
---
## Command

Use mkimage
```
$ sudo mkimage -A arm64 -T script -C none -d boot.cmd boot.scr
```
<br>

<span style="{{ site.code }}">boot.cmd</span> is a bootscript txt file.<br>
<span style="{{ site.code }}">boot.scr</span> is a bootscript of odroids.<br>
