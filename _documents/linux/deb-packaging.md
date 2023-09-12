---
permalink: /documents/linux/deb-packaging/
title: "Debian Packaging"
excerpt: "how to do ubuntu deb packaging"
toc: true
---

## Guide for Debian Packaging

This page is based on the [debmake guide page](https://www.debian.org/doc/manuals/debmake-doc/).<br>

I changed it very briefly.<br>

### Setups

There are some requirements<br>

```
autotools-dev
build-essential
devscripts
debhelper
equivs
```

```
$ sudo apt-get install autotools-dev build-essential devscripts debhelper equivs
```

These are the minimum debian packages required for packaging operations,<br>
and additional packages required can be found [here](https://www.debian.org/doc/manuals/developers-reference/tools.html).<br>

Once you've installed all the tools you need,<br>
set your name and email address.<br>

Let's setup dhese packages by adding the following lines to <span style="{{ site.code }}">~/.bashrc</span> .<br>
```
DEBEMAIL="your_email_address"
DEBFULLNAME="Firstname Lastname"
export DEBEMAIL DEBFULLNAME
```

### Packaging manual

You just follow the structured file tree format.<br>
Create the required directory/file and add the necessary parts.<br>
Let's create a Debian package with a simple example.<br>

### Simple example

I'm going to make a package that prints a hello message when put in the <span style="{{ site.code }}">welcome</span> command.<br>
I will implement hello message in C language with <span style="{{ site.code }}">automake</span> .<br>

You can name the package what you want. I chose <span style="{{ site.code }}">deb-welcome</span> .<br><br>

Make debian package directory
```
$ mkdir -p deb-welcome/debian
$ cd deb-welcome
```
<br><br>

Make package requirements files
```
$ touch debian/changelog && \
  touch debian/compat && \
  touch debian/control && \
  touch debian/copyright && \
  touch debian/deb-welcome.install && \
  touch debian/rules
```
<br><br>

Edit debian/control
```
$ vi debian/control
```
```
Source: deb-welcome
Secstion: misc
Prioriy: optional
Maintainer: Steve Jeong <steve@how2flow.net>
Build-Depends: debhelper (>= 10)

Package: deb-welcome
Architecture: any
Multi-Arch: foreign
Depends: ${misc:Depends}, ${shlibs:Depends}
Description: The Simple example Packages
```
<span style="{{ site.code }}">debian/control</span> contains the most vital information about the source package and about the binary packages it creates.<br>
It is in key-value format, and see [feilds](https://www.debian.org/doc/debian-policy/ch-controlfields.html#source-package-control-files-debian-control) each attribute value.<br><br>

Edit debian/changelog
```
$ vi debian/changelog
```
```
deb-welcome (0.0.1) unstable; urgency=medium

  * initial release.

 -- Steve Jeong <steve@how2flow.net>  Fri, 19 May 2023 14:13:25 +0900
```
<span style="{{ site.code }}">debian/changelog</span> is a release note.<br>
If you improved/modified the package and there is no change in changelog, the package will not be built.<br>
After the first version, you can update <span style="{{ site.code }}">debian/changelog</span> with <span style="{{ site.code }}">dch -i</span> .<br><br>

Edit debian/copyright
```
$ vi debian/copyright
```
```
deb-welcome is Copyright (C) 2023 Steve Jeong

deb-template is free software; you can redistribute it and/or modify it
under the terms of the GNU General Public License as published
by the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.

You should have received a copy of the GNU General
Public License along with this program.
If not, see <https://www.gnu.org/licenses/>.
```
licenses is in <span style="{{ site.code }}">/usr/share/common-licenses/</span> .<br>
I apply the <span style="{{ site.code }}">GPL3 license</span> .<br>
The parent path should also indicate license use.
```
$ sudo cp /usr/share/common-licenses/GPL-3 ../LICENSE
```
<br><br>

Edit debian/compat
```
$ vi debian/compat
```
```
13
```
The compat file defines the debhelper compatibility level.<br>
You may use compat level <span style="{{ site.code }}">9</span> in certain circumstances for compatibility with older systems.<br>
However, using any level below <span style="{{ site.code }}">9</span> is not recommended and should be avoided for new packages.<br>

In <span style="{{ site.code }}">debian/control</span>, debhelper version is <span style="{{ site.code }}">>= 10</span>, must be at least 10.<br>
I'm going to use <span style="{{ site.code }}">autoconf/automake</span>, so I raise the value more.<br><br>

Edit debian/rules
```
$ vi debian/rules
```
```
%:
	dh $@ --with autoreconf
```
<span style="{{ site.code }}">debian/rules</span> is the packaging rule file.<br>
<span style="{{ site.code }}">$@</span> is one of shell macro. The details will not be covered here.<br><br>

Then, the package file-tree is:
```
deb-welcome
|
`debian
|`changelog
| compat
| control
| copyright
| rules
|
`LICENSE
```
If you have created debian package configuration files, you should now create a file for the <span style="{{ site.code }}">automake<span> build.<br><br>

Autoscan
```
$ autoscan
$ ls
autoscan.log  configure.scan
$ mv configure.scan configure.ac
```

```
$ vi configure.ac
```
```
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([debwelcome], [3.0], [steve@how2flow.net])
AC_CONFIG_SRCDIR([Makefile.am])
AC_CONFIG_HEADERS([config.h])
AC_SUBST([EXTRA_CFLAGS], "-Wall -Werror")

AM_INIT_AUTOMAKE([foreign])

# Checks for programs.
AC_PROG_CC
AC_PROG_INSTALL

# Checks for macros.
AC_CONFIG_MACRO_DIRS([m4])

# Checks for library functions.
AC_CONFIG_FILES([
  Makefile
  src/Makefile])

AC_OUTPUT
```

Make C source
```
$ vi Makefile.am
```
```
ACLOCAL_AMFLAGS = -I m4 ${ACLOCAL_FLAGS}

SUBDIRS = src
```
```
$ mkdir -p src
```
```
$ vi src/main.c
```
```
#include <stdio.h>

int main(int argc, char *argv[])
{
	printf("hello???\n");
	return 0;
}
```
```
$ vi src/Makefile.am
```
```
bin_PROGRAMS = welcome

welcome_SOURCES = main.c
```
<br>

Make autoreconf script
```
$ vi autogen.sh
```
```
#!/bin/sh

autoreconf -ivf || exit 1
```
<br>

Then, the package file-tree is:
```
deb-welcome
|
`debian/
|`changelog
| compat
| control
| copyright
| rules
|
`src/
|`main.c
| Makefile.am
|
`autogen.sh
 autoscan.log
 configure.ac
 LICENSE
```
<br>

All we have left is the package build and installation.<br>
Let's add a file and proceed with the build and installation.<br><br>

Package Install
```
$ ./autogen.sh
$ ./configure
$ make
$ sudo make install
```
<br>

Package test
```
$ welcome
hello???
```

#### Output deb file

When you create a Debian file,<br>
You can add commands <span style="{{ site.code }}">before-during-after</span> installing and <span style="{{ site.code }}">before-during-after</span> removing a package.<br>
It exists in the form of a script or data field.<br>

<span style="{{ site.code }}">debian/"package_name".install</span> is data feild file.<br>
Not only install, but also <span style="{{ site.code }}">preinst</span> , <span style="{{ site.code }}">postinst</span> , <span style="{{ site.code }}">prerm</span> , and <span style="{{ site.code }}">postrm</span> .<br>

Let's just add the install file.<br><br>

Make 'install' script
```
$ mkdir -p etc/share/deb-welcome
$ vi etc/share/deb-welcome/welcome_docs
```
```
hello??? my name is steve
```
```
$ vi debian/deb-welcome.install
```
```
etc/share/deb-welcome/welcome_docs /usr/share/deb-welcome/
```

<span style="{{ site.code }}">xxx.install</span> copies the first field to the second field. <span style="{{ site.code }}">wildcards</span> are also available.<br>
They exist in script form and operate before and after the command upon installation or removal of the package.<br>

I will now create & install the deb package.<br><br>

Create & install debian package file
```
$ sudo make uninstall
$ debuild -uc -us -i -b
$ sudo dpkg -i ../deb-welcome_0.0.1_amd64.deb
```

Check
```
$ welcome
hello???
$ cat /usr/share/deb-welcome/welcome_docs
hello??? my name is steve
```

Finally, debian package [example code](https://github.com/how2flow/kernel-scripts) that is released and used package.<br>

#### Build External Package Repository

If you need to download the package repository and build it locally,<br>
The local environment must have all the <span style="{{ site.code }}">Build-Depends</span> packages installed in the <span style="{{ site.code }}">debian/control</span> file.<br>

Use <span style="{{ site.code }}">mk-build-deps</span> which is part of <span style="{{ site.code }}">devscripts</span> .<br>

### Package Deployment

how to deploy debian package?<br>
Normally, you have to upload it to Debian server, <br>
You have to be a debian contributor first and the process is cumbersome.<br>

So, many people are use PPA.<br>
PPA is **P**ersonal **P**ackage **A**rchive.<br>
[Launchpad](https://launchpad.net) is a representative platform for supporting PPA.<br>
