---
categories:
- linux
excerpt: This post covers how to use Make and CMake, which are build automation tools
  in Linux.
header:
  teaser: /assets/images/posts/thumbnails/note.jpg
layout: single
tags:
- build
- cmake
- compile
- debian
- languages
- make
- ubuntu
title: '[Linux] Make and CMake'
toc: true
redirect_from:
- /documents/linux/build-scripts/
- /legacy/build-scripts/
- /documents/linux/build-scripts
- /legacy/build-scripts
---
## Introduction

Linux offers various build tools.<br>
In this post, I will discuss <span style="{{ site.code }}">Make</span> and <span style="{{ site.code }}">CMake</span>, two of the most widely used tools.<br>
These are build automation tools commonly used in C projects.<br>

While <span style="{{ site.code }}">Automake</span> is also frequently used for project packaging,<br>
I have covered that in a separate post on packaging and will write more if needed.<br>

## Make

### Installing make

<span style="{{ site.code }}">make</span> is a build automation tool that can be installed as follows:
```
$ sudo apt install build-essential # Debian
```
```
$ sudo dnf install make # Redhat
```

### make Example

I will use a build example using the <span style="{{ site.code }}">gcc</span> compiler.<br>
Ultimately, <span style="{{ site.code }}">make</span> executes the following command:
```
$ gcc -o app.out app.c
```
<br>

However, if most projects consisted of just one source file and one executable,<br>
a build tool wouldn't be necessary.<br>
<span style="{{ site.code }}">make</span> allows you to organize numerous source files and complex build options into a single script.<br>
The make command is executed by substituting it with commands based on the contents of the <span style="{{ site.code }}">Makefile</span> script.<br>

Writing the gcc command as a <span style="{{ site.code }}">Makefile</span> looks like this:
```
main:
	gcc -o app.out app.c
```
Here, the command line must be indented with a tab for the <span style="{{ site.code }}">Makefile</span> to recognize it as a command.<br>
However, as you encounter various projects, you'll realize that <span style="{{ site.code }}">Makefile</span>s are rarely this simple.<br>

Here is an example of a file structure and a <span style="{{ site.code }}">Makefile</span>.<br>

File-Tree:
```
.
|-bar.c
|-build/
|-foo.c
|-include/
|-Makefile
```

Makefile:
```
 1 build_dir = $(PWD)/build
 2 include_dirs = $(PWD)/include
 3 srcs = $(wildcard ./*.c)
 4 objs = $(patsubst %.c, $(build_dir)/%.o, $(srcs))
 5
 6 CC = gcc 
 7 CFLAGS = -g -I$(include_dirs)
 8 LDFLAGS = 
 9 TARGET = main
10
11 # functions
12 all: $(TARGET)
13
14 $(TARGET): $(objs)
15	@$(CC) -o $@ $(objs) $(CFLAGS) $(LDFLAGS)
16
17 $(build_dir)/%.o: %.c
18	@mkdir -p $(shell dirname $@)
19	@$(CC) -c -o $@ $< $(CFLAGS) $(LDFLAGS)
20
21 clean:
22	@rm -rf $(build_dir) $(TARGET)
```
Let's assume we have a project like this.<br>
It might look complex, but it's not difficult if you examine it piece by piece.<br>
In reality, there are many projects much more complex than this.<br>

Lines 1–9 define the script's variables.<br>
Several <span style="{{ site.code }}">Makefile</span> functions are used in the variable definition section.<br>

The <span style="{{ site.code }}">wildcard</span> on line 3 means using Linux wildcards.<br>
Linux wildcards include <span style="{{ site.code }}">*</span>, <span style="{{ site.code }}">?</span>, <span style="{{ site.code }}">\ </span>, <span style="{{ site.code }}">^</span>, etc.<br> Please refer to [Linux wildcard](https://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm) for their respective meanings.<br>

In the example, <span style="{{ site.code }}">\$(wildcard ./*.c)</span> is used,<br>
which targets all files ending in .c, following the Linux wildcard convention.<br>
Given the file structure above, the <span style="{{ site.code }}">srcs</span> variable is defined as follows:
```
srcs = foo.c bar.c
```

Line 4 contains the following expression:<br>
<span style="{{ site.code }}">patsubst</span><br>
<br>

Before discussing <span style="{{ site.code }}">patsubst</span>, let's look at <span style="{{ site.code }}">subst</span>.<br>
<span style="{{ site.code }}">subst</span> is short for substitution.<br>
In other words, it replaces text.<br>

The format is as follows:
```
$(subst from,to,target)
```

For example,
```
$(subst ple,PLE,apple)
```
The result is apPLE.<br>
<br>

Now for <span style="{{ site.code }}">patsubst</span>.<br>
<span style="{{ site.code }}">patsubst</span> stands for pattern substitution, meaning it replaces based on a pattern.<br>
The format is the same, but it requires a pattern instead of plain text.<br>
The meanings of <span style="{{ site.code }}">*</span> and <span style="{{ site.code }}">%</span> used in the example are identical.<br>
According to the <span style="{{ site.code }}">Makefile</span>, all source files in the current directory are mapped to the build directory with their extension changed to<br>
<span style="{{ site.code }}">.o</span> (this defines the variables; it doesn't create the actual files yet).<br>
Based on the <span style="{{ site.code }}">srcs</span> variable, the <span style="{{ site.code }}">objs</span> variable is defined as follows:<br>
```
objs = $(patsubst %.c, $(build_dir)/%.o, foo.c bar.c)
```
```
objs = build/foo.o build/bar.o
```
<br>

The variable section can be simplified as follows:
```
 1 build_dir = ./build
 2 include_dirs = ./include
 3 srcs = "foo.c bar.c"
 4 objs = "build/foo.o build/bar.o"
 5
 6 CC = gcc 
 7 CFLAGS = -g -I./include
 8 LDFLAGS = 
 9 TARGET = main
10
11 # functions
12 all: main
13
14 main: build/foo.o build/bar.o
15 	@gcc -o $@ build/foo.o build/bar.o -g -I./include
16
17 build/foo.o build/bar.o: %.c
18 	@mkdir -p $(shell dirname $@)
19 	@gcc -c -o $@ $< -g -I./include
20
21 clean:
22 	@rm -rf ./build main
```

Now for the command section.<br>

<span style="{{ site.code }}">all: main</span> ,<br>
<span style="{{ site.code }}">main: build/foo.o build/bar.o</span> ,<br>
<span style="{{ site.code }}">build/foo.o build/bar.o: %.c</span> ,<br>

The <span style="{{ site.code }}">A:B</span> format in the command section represents Target:Dependency.<br>

On line 12, all represents the entire command to be executed in the <span style="{{ site.code }}">Makefile</span> (<span style="{{ site.code }}">$ make all</span>) and requires main.<br>
The main target called by all is defined on line 14.<br>
main depends on build/foo.o and build/bar.o.<br>
build/foo.o and build/bar.o are defined on line 16.<br>

First, the <span style="{{ site.code }}">@</span> symbol at the beginning of each command means the command itself won't be echoed to the terminal.<br>
The <span style="{{ site.code }}">$@</span> used on lines 15, 18, and 19 refers to the Target A in the <span style="{{ site.code }}">A:B</span> relationship,<br>
and <span style="{{ site.code }}">$<</span> on line 19 refers to the Dependency B.<br>

To summarize once more:
```
 1 build_dir = ./build
 2 include_dirs = ./include
 3 srcs = "foo.c bar.c"
 4 objs = "build/foo.o build/bar.o"
 5
 6 CC = gcc 
 7 CFLAGS = -g -I./include
 8 LDFLAGS = 
 9 TARGET = main
10
11 # functions
12 all: main
13
14 main: build/foo.o build/bar.o
15 	@gcc -o main build/foo.o build/bar.o -g -I./include
16
17 build/foo.o build/bar.o: %.c
18 	@mkdir -p build
19 	@gcc -c -o build/foo.o build/bar.o foo.c bar.c -g -I./include
20
21 clean:
22 	@rm -rf ./build main
```

<span style="{{ site.code }}">dirname</span> extracts the directory name, so it was substituted with build.<br>
The <span style="{{ site.code }}">Makefile</span> is executed in the following order:<br>

```
make -> make all -> main -> build/foo.o build/bar.o -> mkdir -p build -> gcc -c -o build/foo.o build/bar.o foo.c foo.o -g -I ./include -> gcc -o main build/foo.o build/bar.o -g -I ./include
```

## CMake

Like make, CMake is a build automation tool.<br>
It is much more intuitive and easier to write than make.<br>
Regular updates provide a diverse and powerful API.<br>
CMake scripts are written in <span style="{{ site.code }}">CMakeLists.txt</span>.

### Installing CMake

Install cmake using the following command:
```
$ sudo apt install cmake # Debian
```
```
$ sudo yum install cmake # Redhat
```
<br>

### CMake Example

I will use the same project structure and compiler (gcc) as in the make example.<br>

File-Tree:
```
.
|-bar.c
|-build/
|-foo.c
|-include/
|-CMakeLists.txt
```
<br>

CMakeLists.txt:
```
 1 cmake_minimum_required(VERSION 3.5)
 2 project(projectname)
 3
 4 # set vars
 5 set(SRC_FILES foo.c bar.c)
 6
 7 # print
 8 message(${CMAKE_PROJECT_NAME})
 9 message(${SRC_FILES})
10
11 # global options
12 add_compile_options(-g -Wall ... )
13 add_definitions(-DFLASH -DMACRO ...)
14 include_directories(include) # like compile option '-I'
15 link_directories(...) # like compile option '-L'
16 link_libraries(opencv rga samba ...) # like compile option '-l' (libopencv.so / librga.so / libsamba.so) ...
17
18 # target options
19 add_excutable(app.out ${SRC_FILES})
10 add_library(test [STATIC|SHARED|MODULE] foo.c bar.c) # output: libtest.a / libtest.so / test.cmake
21 add_dependencies(flash app.out) # "Target-to-Target" dependency. 'flash' <- 'app.out'
22
23 # specific target options
24 target_compile_options(app.out PUBLIC -g -wall)
25 target_conpile_definitions(app.out PUBLIC -DDEVMEM)
26 target_include_directories(app.out PUBLIC include driver/include)
27 target_link_libraries(app.out test) # use "-static" only use archive
28
29 # install
30 install(TARGETS app.out
31         RUNTIME_DESTINATION /usr/local/bin
32         LIBRARY_DESTINATION /usr/local/lib
33         ARCHIVE_DESTINATION /usr/share/app
34 )
```

Unlike <span style="{{ site.code }}">Makefile</span>, which requires writing shell commands directly, CMake has its own API.<br>
The <span style="{{ site.code }}">global options</span> identified in the comments apply to the entire project,<br>
while <span style="{{ site.code }}">target options</span> are limited to the target specified in the parameters.<br>

#### Commonly Used CMake Variables

Below are some frequently used variables in CMakeLists.txt:<br>

<span style="{{ site.code }}">CMAKE_VERBOSE_MAKEFILE</span> value: true/false<br>
<span style="{{ site.code }}">true</span>: Prints build logs.<br>
<span style="{{ site.code }}">false</span>: Does not print build logs.<br>
<br>
<span style="{{ site.code }}">CMAKE_BUILD_TYPE</span> value: Release/Debug/RelWithDebinfo/MinSizeRel<br>
<br>
<span style="{{ site.code }}">CMAKE_C_FLAGS_{CMAKE_BUILD_TYPE}</span> value: cflags<br>
e.g.
```
set(CMAKE_C_FLAGS_RELEASE "-DCONFIG_RELEASE -03")
```
<br>
<span style="{{ site.code }}">CMAKE_EXE_LINKER_FLAGS_{CMAKE_BUILD_TYPE}</span> value: lflags<br>
e.g.
```
set(CMAKE_EXE_LINKER_FLAGS_RELEASE "-Wl,-whole-archive")
```
<br>
<span style="{{ site.code }}">DESTINATION</span> value: path default: ${CMAKE_INSTALL_PREFIX}<br>

... More will be added.<br>
