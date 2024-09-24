---
up: "[[CA 1 A Tour of Computer Systems]]"
related:
  - "[[CA 1.5~1.6 Caches Matter]]"
tags:
  - 📝/🌱️
aliases: 
cssclasses:
  - dashboard
created: 2024-09-22T21:14
modified: 2024-09-23T00:32
---
> [!CAUTION] 
> 모든 사진의 출처는 [Computer System: A Programmer's Perspective]에 있음을 밝힙니다. 
## 1.4 Processors Read and Interpret Instructions Stored in Memory

```shell
linux> ./hello
hello, world
linux>
```

- Shell : Unix Application Program & Command-line interpreter
	- Command line 입력을 받아 command를 수행한다. 
	- Command가 아니라면 실행 파일의 이름으로 가정하고 파일을 실행한다.
	- 그러므로 위 코드는 hello라는 실행 파일을 실행하라는 명령어로, hello, world를 출력하게 된다.

![[CA 1. A Tour of Computer Systems-1726995577605.jpeg]]

### 1.4.1 Hardware Organization of a System

#### 버스(Bus)
- 시스템 전체에서 컴포넌트 간에 바이트 정보를 주고 받는 전기 전선의 모음
- 일반적으로 단어라는 고정 크기의 바이트 뭉치(chunk)로 전송되도록 설계되어 있다.
- 고정 크기 바이트는 시스템 마다 달라지는 매개변수이다.
- 최신 대부분 기계는 4바이트(32비트) 또는 8바이트(64비트)의 단어 크기를 갖는다.

#### 입출력 장치(I/O Devices)
- 시스템과 외부 세계의 연결 역할을 한다. ex) 키보드, 마우스, 디스플레이, 디스크 등
- 각 입출력 장치는 컨트롤러(controller) 혹은 어댑터(adapter)에 의해 I/O 버스와 연결된다. 컨트롤러와 어댑터의 차이는 주로 패키징(packaging)에 의해 발생한다.
	- 컨트롤러 : 장치 자체 또는 시스템의 메인 회로 기판(마더보드, motherboard)에 있는 칩셋
	- 어댑터 : 마더보드 슬롯에 꽃히는 카드

#### 주 메모리/메인 메모리(Main Memory)
- 프로세서가 프로그램을 실행하는 동안 조작하는 데이터와 프로그램을 저장하는 임시 저장 장치
- 물리적으로, 메인 메모리는 DRAM 칩의 모음으로 구성된다.
- 논리적으로, 메모리는 고유한 주소(인덱스)를 가진 바이트의 선형 배열로 구성된다.


#### 프로세서(Processor)
- 중앙 처리 장치(CPU)를 뜻한다.
- 전원이 켜지고 꺼질 때까지 메인 메모리에 저장된 명령어(instruction)를 해석/실행 하는 엔진이다. 
- 프로그램 카운터(PC)라는 단어 크기 저장 장치/레지스터(register)가 핵심으로 작동한다.
	- PC는 현재 실행될 명령어의 주소를 저장하여 프로세서가 올바른 순서로 명령을 실행하게 한다.
- 프로세서는 다음과 같은 과정으로 실행된다.
	- 프로세서는 반복적으로 PC가 가리키는 메모리 주소에서 명령어를 읽고 명령어 비트를 해석한다.
	- 명령어에 의해 지시된 작업을 수행한다. 
	- PC를 업데이트하여 다음 명령어를 가리킨다.
	- (다음 명령어는 이전에 실행된 명령어와 메모리에서 연속적일 수도 있고, 아닐 수도 있다.)
- 주로 메인 메모리, 레지스터 파일, 산술/논술 유닛(ALU)을 중심으로 동작한다.
	- 레지스터 파일
	    - 여러 개의 고유한 이름을 가진 레지스터로 구성된 작은 저장 장치.
	    - 각 레지스터는 데이터나 주소 값을 임시로 저장한다.
	- 산술/논리 장치(ALU)
	    - 새로운 데이터 및 주소 값을 계산한다.
- 프로세서가 명령어 요청에 따라 수행할 수 있는 작업의 간단한 예는 다음과 같다.
	- Load : 메인 메모리 -> 레지스터로 바이트 또는 단어를 복사해 레지스터의 이전 내용을 덮어쓴다.
	- Store: 레지스터 -> 메인 메모리로 바이트 또는 단어를 복사해 그 위치의 이전 내용을 덮어쓴다.
	- Operate: 두 레지스터의 내용을 ALU로 복사하고, 두 단어에 대해 산술 연산을 수행하여 결과를 레지스터에 저장하며, 그 레지스터의 이전 내용을 덮어쓴다.
	- Jump: 주어진 주소를 PC에 복사해 프로그램의 실행 흐름을 변경한다. 현재 PC 값을 덮어쓴다.

> [!NOTE] [[DRAM, SRAM, ALU]]

### 1.4.2 Running the `hello` Program

![[CA 1. A Tour of Computer Systems-1727005962779.jpeg]]
![[CA 1. A Tour of Computer Systems-1727006816588.jpeg]]![[CA 1. A Tour of Computer Systems-1727006828032.jpeg]]

1. Shell 프로그램이 입력을 기다린다.
2. 키보드에 `./hello`를 입력할 때 shell은 각각의 문자를 레지스터로 읽고 메모리에 저장한다.
3. 사용자가 엔터 키를 누르면 shell은 명령어 입력을 마쳤음을 인식한다.
4. Shell은 `hello`실행 파일을 디스크에서 메인 메모리로 불러오기 위해 코드와 데이터("hello, world\\n")를 복사하는 일련의 명령어를 실행한다.
	- DMA(Direct Memory Access)를 통해 데이터는 프로세서를 거치지 않고 디스크에서 메인 메모리로 이동한다. (Figure 1.6 참고)
5. `hello`파일의 코드와 데이터가 메모리에 로드되면 프로세서는 `hello` 파일의 기계어 명령을 실행한다.
6. 기계어 명령을 통해 메모리에 있는 "hello, world\\n" 문자열 바이트를 레지스터 파일로 복사한다.
7. 레지스터 파일 내용을 디스플레이로 전송한다. (Figure 1.7 참고)