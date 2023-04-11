---
permalink: /posts/build-scripts/
title: "[Linux] Make와 CMake"
excerpt: "리눅스의 빌드 자동화 도구인 Make와 Cmake 사용 포스팅 입니다."
header:
  teaser: /assets/posts/images/note.jpg
categories:
  - Linux
tags:
  - build
  - cmake
  - compile
  - debian
  - languages
  - make
  - ubuntu
toc: true
---

## 소개

리눅스에는 다양한 빌드 도구들이 있습니다.<br>
그 중에서 가장 많이 사용하는 <span style="{{ site.code }}">Make</span> 와 <span style="{{ site.code }}">CMake</span> 에 대해 포스팅 합니다.<br>
이 도구들은 보통 C 프로젝트에서 많이 사용하는 빌드 자동화 도구 입니다.<br>

프로젝트 패키징에 <span style="{{ site.code }}">Automake</span> 도 많이 사용하지만,<br>
관련 내용은 패키징 포스팅에서 다뤘었고 더 필요하다면 따로 작성하겠습니다.<br>

## Make

### make 설치하기

<span style="{{ site.code }}">make</span> 는 빌드 자동화 도구로, 다음과 같이 설치할 수 있습니다.
```
$ sudo apt install build-essential # Debian
```
```
$ sudo dnf install make # Redhat
```

### make 예제

<span style="{{ site.code }}">gcc</span> 컴파일러를 사용한 빌드로 예시를 들겠습니다.<br>
<span style="{{ site.code }}">make</span> 는 결론적으로 다음 명령을 실행합니다.
```
$ gcc -o app.out app.c
```
<br>

하지만 대부분 프로젝트가 소스파일 하나, 실행파일 하나로 끝이난다면,<br>
빌드 도구가 따로 필요 없을 것입니다.<br>
수많은 소스파일들과 복잡한 빌드 옵션들을 하나의 스크립트로 정리할 수 있는 것이 <span style="{{ site.code }}">make</span> 입니다.<br>
make 명령은 <span style="{{ site.code }}">Makefile</span> 스크립트 내용을 기반으로 하나의 명령어로 치환되어 실행됩니다.<br>

gcc 커맨드를 <span style="{{ site.code }}">Makefile</span>로 작성하면 다음과 같습니다.
```
main:
	gcc -o app.out app.c
```
여기서 명령줄은 반드시 맨 앞에 tab 으로 구분지어야 <span style="{{ site.code }}"><span style="{{ site.code }}">Makefile</span></span> 에서 명령으로 인식합니다.<br>
그러나 여러 프로젝트들을 접하다 보면 알겠지만, <span style="{{ site.code }}"><span style="{{ site.code }}">Makefile</span></span> 은 결코 단순하지 않습니다.<br>

파일 구조와 <span style="{{ site.code }}">Makefile</span> 의 예시 입니다.<br>

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
이와 같은 프로젝트가 있다고 가정해 봅시다.<br>
꽤 복잡해 보여도, 하나씩 살펴보면 어렵지 않습니다.<br>
실제로 이보다 훨씬 복잡한 프로젝트가 더 많이 있습니다.<br>

1~9 번째 라인은 스크립트의 변수를 정의합니다.<br>
변수 정의하는 부분에서 <span style="{{ site.code }}">Makefile</span>의 여러 함수가 사용되었습니다.<br>

3번 라인의 <span style="{{ site.code }}">wildcard</span> 은 리눅스의 와일드카드를 사용하겠다는 뜻입니다.<br>
리눅스의 와일드카드는, <span style="{{ site.code }}">\*</span> , <span style="{{ site.code }}">?</span> , <span style="{{ site.code }}">\ </span> , <span style="{{ site.code }}">^</span> 등이 있습니다.<br> 각각의 의미는 [Linux wildcard](https://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm) 참조 바랍니다.<br>

예제 파일에서는 <span style="{{ site.code }}">\$(wildcard ./\*.c)</span> 가 사용되었는데,<br>
리눅스 와일드카드 의미대로, .c 로 끝나는 모든 파일이 대상이 됩니다.<br>
위의 파일구조를 가지고 있다면, 변수 <span style="{{ site.code }}">srcs</span> 은 다음과 같이 정의 됩니다.
```
srcs = foo.c bar.c
```

4번 라인에서 다음 표현이 있습니다.<br>
<span style="{{ site.code }}">patsubst</span><br>
<br>

<span style="{{ site.code }}">patsubst</span> 이전에 <span style="{{ site.code }}">subst</span> 부터 확인하고 넘어가겠습니다.<br>
<span style="{{ site.code }}">subst</span> 은 substitution의 줄임말입니다.<br>
즉, 문자를 치환하는 것입니다.<br>

포멧은 이렇게 작성됩니다.
```
$(subst from,to,target)
```

예시를 들면,
```
$(subst ple,PLE,apple)
```
결과는 apPLE 이 됩니다.<br>
<br>

이제 <span style="{{ site.code }}">patsubst</span> 입니다.<br>
<span style="{{ site.code }}">patsubst</span> 은 pattern substitution, 즉 패턴을 치환하는 것입니다.<br>
포멧은 동일하지만 텍스트가 아닌 패턴이 필요합니다.<br>
예제에서 사용된 <span style="{{ site.code }}">\*</span> 와 <span style="{{ site.code }}">%</span> 의 의미는 동일합니다.<br>
<span style="{{ site.code }}">Makefile</span> 내용대로라면, 현재 디렉토리에 있는 모든 소스파일들이 build 디렉토리 아래에 확장자만<br>
<span style="{{ site.code }}">.o</span> 로 변경된(실제 파일이 생성된 것이 아니라, 변수만 정의된) 상태입니다.<br>
<span style="{{ site.code }}">srcs</span> 변수의 내용을 토대로, 변수 <span style="{{ site.code }}">objs</span> 는 다음과 같이 정의됩니다.<br>
```
objs = $(patsubst %.c, $(build_dir)/%.o, foo.c bar.c)
```
```
objs = build/foo.o build/bar.o
```
<br>

변수 부분을 정리해서 다음과 같이 표현할 수 있습니다.
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

이제 커맨드 부분입니다.<br>

<span style="{{ site.code }}">all: main</span> ,<br>
<span style="{{ site.code }}">main: build/foo.o build/bar.o</span> ,<br>
<span style="{{ site.code }}">build/foo.o build/bar.o: %.c</span> ,<br>

이와 같이 명령 부분의 <span style="{{ site.code }}">A:B</span> 형식은 Target:Dependence 입니다.<br>

12번 라인에서 all은 <span style="{{ site.code }}">Makefile</span>에서 실행할 명령어 전체( <span style="{{ site.code }}">$ make all</span> )를 나타내고, main을 필요로 합니다.<br>
all 에서 호출하는 main은 14번 라인에서 정의되어 있습니다.<br>
main은 build/foo.o 와 build/bar.o를 필요로 합니다.<br>
build/foo.o 와 build/bar.o는 16번 라인에서 정의되어 있습니다.<br>

먼저, 각각의 명령어에서 공통으로 사용된 제일 앞의 <span style="{{ site.code }}">@</span> 은 명령어를 echo 하지 않겠다는 의미입니다.<br>
15, 18, 19번 라인에서 사용된 <span style="{{ site.code }}">$@</span> 은 <span style="{{ site.code }}">A:B</span> 에서 Target인 A를 뜻하고,<br>
19번 라인에서 사용된 <span style="{{ site.code }}">$<</span> 은 Dependence인 B를 나타냅니다.<br>

다시 한번 정리하자면,
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

<span style="{{ site.code }}">dirname</span>은 디렉토리 이름을 추출하기 때문에 build로 치환되었습니다.<br>
<span style="{{ site.code }}">Makefile</span>은 다음과 같이 실행됩니다.<br>

```
make -> make all -> main -> build/foo.o build/bar.o -> mkdir -p build -> gcc -c -o build/foo.o build/bar.o foo.c foo.o -g -I ./include -> gcc -o main build/foo.o build/bar.o -g -I ./include
```

## CMake

CMake 역시 make와 마찬가지로 빌드 자동화 도구 입니다.<br>
make보다 훨씬 직관적이고 작성이 편합니다.<br>
꾸준한 업데이트로 다양하고 강력한 API를 제공합니다.<br>
CMake 스크립트는 <span style="{{ site.code }}">CmakeLists.txt</span> 으로 작성합니다.

### CMake 설치하기

다음 명령어를 통해 cmake를 설치합니다.
```
$ sudo apt install cmake # Debian
```
```
$ sudo yum install cmake # Redhat
```
<br>

### CMake 예제

앞서 make와 동일한 구조의 프로젝트와 동일한 컴파일러(gcc)를 예시로 설명하겠습니다.<br>

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

CmakeLists.txt:
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

shell 명령을 직접 작성해야 했던 <span style="{{ site.code }}">Makefile</span> 과는 다르게, cmake는 자체 API가 있습니다.<br>
주석에서 구분한 <span style="{{ stie.code }}">global options</span> 는 전체에 해당하는 내용이고,<br>
<span style="{{ stie.code}}">target options</span> 는 파라미터에 작성된 타겟 한정입니다.<br>

#### Cmake 자주 사용하는 변수들

다음은 CMakeLists.txt 에서 자주 사용하는 변수들 입니다.<br>

<span style="{{ site.code }}">CMAKE_VERBOSE_MAKEFILE</span> value: true/false<br>
<span style="{{ site.code }}">true</span> : 빌드 로그를 출력합니다.<br>
<span style="{{ site.code }}">flase</span> : 빌드 로그를 출력하지 않습니다.<br>
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

... 계속 추가 예정입니다.<br>
