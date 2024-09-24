---
permalink: /documents/trace32-base/
title: Trace32 basic Usage
excerpt: "Generic Interrupt Controller"
comments: false
toc: true
---

## Start Trace32 with cmm script

Trace32 디버거를 사용할 때, 타겟보드에 연결하기 위한 cmm 스크립트를 작성한다.<br>
사전에 작성한 cmm 스크립트를 다음 명령어로 불러온다.
```
B:: pedit *
```
<br>

cmm 스크립트에는 다음과 같은 내용들이 있을 것이다.<br>
명령어는 대소문자가 섞여있는데, <U>명령어 자체는 대소문자를 <B>구분하지 않는다.</B></U><br>
다만 대문자로 표현된 부분은 그 명령어의 약어로써 사용가능하다.<br>
(e.g Data.Set은 d.s 로 표현 가능)
```
B::

SYStem.MODE Down
SYStem.CPU CortexA72
SYStem.Reset

SYStem.CONFIG.CoreNumber 1
SYStem.CONFIG.COREBASE DAP:0x80010000
SYStem.CONFIG.CTIBASE DAP:0x80012000

CORE.NUMber 1

SYStem.Option.EnReset OFF
SYStem.Option.CFLUSH OFF

TrOnchip.Set RESET OFF

SYStem.Mode Attach

if run()
(
	BREAK
	GOSUB DEACTIVATE_WDT
	GOSUB UPLOAD_ELF
)

print "Attached"
ENDDO

DEACTIVATE_WDT:
	Data.Set AHD:0x15000000 %LE %Long 0x0
	Data.Set AHD:0x15001000 %LE %Long 0x0

UPLOAD_ELF:
	d.load.elf Z:\linux\vmlinux. /nocode
	y.SourcePATH.Translate "\home\how2flow" "Z:"
```
우선 실행환경은 Window OS에서 PowerView를 실행한다고 가정한다.<br>
(Ubuntu나 기타 Linux 기반 OS에서 PowerView를 지원하는지는 모르겠다.)<br>
<br>
예를 들어 위와 같은 cmm 스크립트가 있다고 한다면,<br>
CPU는 <span style="{{ site.code }}">CortexA72</span> 이고 Core는 1개를 연결했다.<br>
당연히 2개 이상 연결도 가능하다.<br>
연결할 코어 개수 만큼 <span style="{{ site.code }}">SYStem.CONFIG.CoreNumber 4</span> 이런 식으로 써주면 된다.<br>
<span style="{{ site.code }}">CORE.NUMber</span> 역시 숫자를 맞춰줘야 한다.<br>
<br>

<span style="{{ site.code }}">SYStem.CONFIG.COREBASE</span> 와 <span style="{{ site.code }}">SYStem.CONFIG.CTIBASE</span> 은<br>
반드시 설정해야 한다.<br>
연결할 코어의 디버그 포트 주소를 <span style="{{ site.code }}">DAP::</span> 접두사를 붙여 추가해 준다.<br>
Debug Access Port 라는 뜻으로, 각 코어마다 해당 포트 주소가 존재하며,<br>
이는 해당 타겟 보드의 데이터시트나 TRM을 참조하면 된다.<br>
<br>

추가로 반드시 작업해 주어야 하는것이 WDT 비활성화 이다.<br>
Trace32는 Runtime 중에 Break를 걸어서 데이터를 확인하고 디버깅 하기 때문에,<br>
WDT에서 설정한 시간동안 Break가 걸려 있다면 강제로 Reset되고 디버거 연결이 끊어진다.<br>
WDT비활성화 역시 타겟보드의 데이터시트나 TRM을 참조하면 된다.<br>

## Dump registers binary

Trace32 Powerview의 cmdline에 다음을 입력하면<br>
현재 프로그램의 실행경로를 알 수 있다.
```
B:: cd
```
<br>

특정 주소의 레지스터 데이터를 저장하고 싶으면 다음과 같은 명령어를 사용한다.<br>
역시 대문자는 약어로 표현 가능하다.
```
B:: Data.SAVE.Binary <filename> <address>
```
```
B:: D.SAVE.B <filename> <address>
```
```
B:: d.SAVE.b <filename> <address>
```
<br>

레지스터 덤프 범위는 다음과 같이 정할 수 있다.<br>
('++offset' 을 쓰면 1바이트 만큼 더 저장된다. 2017 기준)
```
B:: d.SAVE.b C:\T32\tesr_result a:0x405B0000++0x1000 // (+ 4K + 1)
```
<span style="{{ site.code }}">++</span> 는 뒤의 크기만큼 offset으로 범위를 저장하고,<br>
<span style="{{ site.code }}">--</span> 는 뒤에 절대주소가 온다. 뒤에 오는 주소까지 범위로 저장한다.<br>
<br>

특정 레지스터 주소의 데이터 변화를 비교하고 싶을 때,<br>
before after 두가지 레지스터 상태를 덤프하고<br>
<span style="{{ site.code }}">Beyond Compare</span> 프로그램 등으로 비교하면 편리하다.<br>
