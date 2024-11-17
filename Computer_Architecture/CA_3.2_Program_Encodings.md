---
created: 2024-10-02T19:33
modified: 2024-11-17T17:44
---
# 3.2 Program Encodings

- C 프로그램의 2가지 파일을 작성했다고 가정해보자. (p1.c, p2.c)
- 두 파일을 Unix 커맨드라인을 통해서 컴파일하자.

```shell
linux> gcc -Og -o p p1.c p2.c
```

- gcc 커맨드는 C 컴파일러를 의미한다.
	- gcc 컴파일러가 리눅스에서 기본 컴파일러이기 때문에 cc 라는 커맨드로 간단하게 호출할 수 있다.
- -Og : 기존 C 코드의 전체적인 구조에 따라 기계 코드를 생성하는 최적화 수준을 적용하라는 옵션
	- 디버깅 정보를 담는 옵션
	- gcc 버전 4.8에서 도입되었다. 이전 버전의 경우 -O1 을 사용하는 것이 원래 프로그램 구조를 따르는 코드를 생성하는데 좋다.
	- -Og 조건을 붙이지 않고 최고 수준의 최적화를 할 경우 코드가 크게 변형되 읽지 못하는 수준이 될 수 있다.
	- -O1, -O2 의 경우 -Og 보다 더 높은 수준의 최적화를 의미하는데, 높은 수준의 최적화를 진행할 경우 더 높은 성능을 보인다.
- -o p : p 이름의 실행 파일을 만든다.

## 변환 과정

![alt text](<../Assets/Computer_Architecture/CA 1. A Tour of Computer Systems-1726993752388.jpeg>)

- gcc 커맨드는 소스 코드에서 실행 가능한 코드로 변경하는 프로그램의 전체 과정을 호출한다.

1. C 전처리기(Preprocessor)는 소스 코드를 확장해 `#include`로 포함된 모든 파일을 호출하고, `#define`으로 선언된 매크로를 확장한다.
2. 컴파일러는 p1.s, p2.s 라는 이름을 가진 어셈블리 코드 버전의 두 소스 파일을 생성한다.
3. 어셈블러는 어셈블리 소스 코드를 p1.o, p2.o 의 이름을 가진 바이너리 오브젝트 코드(Binary Object Code)로 변환한다.
	- 오브젝트 코드(Object Code)
		- 기계 코드의 첫 번째 형태
		- 모든 명령어에 대해 이진 표현을 포함하지만, 전역값(Global Value)들의 주소는 채워지지 않은 코드이다.
4. 링커는 두 오브젝트 코드 파일과 라이브러리 함수를 구현해놓은 파일을 연결해 최종적으로 실행 가능한 파일 p를 생성한다. 
	- 실행 가능한 코드(Executable Code)
		- 기계 코드의 두 번째 형태
		- 프로세서에 의해 실행되는 코드의 정확한 형태

<br>

---

## 3.2.1 Machine-Level Code

### Abstraction

- 컴퓨터 시스템은 구현 세부 정보를 숨기기 위해 간단한 추상 모델을 사용하여 여러 다양한 추상화 형태들을 사용한다.
- Machine-Level 프로그래밍에서는 추상화에 관하여 2가지가 매우 중요하다.
	1. Machine-Level 프로그램의 형식과 동작은 ISA(Instruction Set Architecture)에 의해 정의된다.
		- 대부분의 ISA는 프로그램의 동작을 명령어가 순차적으로 실행되는 것처럼 설명한다.
		- 실제로는 한 명령어가 끝나기 전에 다음 명령어가 시작된다. 
		- 프로세서 하드웨어의 경우 더 복잡해 동시에 여러 명령어를 실행하나, 순차적으로 실행되는 것과 전체 동작이 일치하도록 보장하는 안전장치를 사용한다.
	2. Machine-Level 프로그램에 의해 사용되는 메모리 주소는 가상 주소(Virtual Addresses)이다.
		- 가상 주소는 매우 큰 바이트 배열로 표현되는 메모리 모델을 제공한다.


### Processor Components

- Program Counter(PC) : 실행되어야 하는 다음 명령어의 메모리 주소를 나타낸다.
	- x86-64에서는 %rip 으로 표현된다.
- 정수 레지스터 파일(Integer Register File) : 64비트 값을 저장할 수 있는 16개의 공간을 포함한다.
	- 레지스터들은 C 포인터에 해당하는 주소 또는 정수 데이터를 가질 수 있다.
	- 일부 레지스터들은 프로그램 상태의 중요한 부분을 추적하는데 사용된다.
	- 나머지 레지스터의 경우 인자, 지역 변수, 함수에 의해 반환되는 값과 같은 일시적인 데이터를 저장하는데 사용된다.
- 조건 코드 레지스터(Condition Code Register) : 최근 실행된 산술 혹은 논리적 명령에 대한 상태 정보를 보관한다.
	- if나 while과 같이 조작이나 데이터 흐름에서 조건적인 변화를 구현하는데 사용된다.
- 벡터 레지스터 집합(Set of Vector Registers) : 하나 이상의 정수, 혹은 부동소수점 값을 저장할 수 있다.

### 메모리 인식에서의 차이

- C 코드 : 데이터 타입에 따라 선언과 메모리 할당을 다양하게 하는 모델을 제공한다.
- 기계 코드 : 메모리를 단순하게 커다란 바이트 주소 저장 배열로 본다.(타입을 구분하지 않는다.)
	- C의 배열, 구조체는 기계 코드에서는 연속적인 바이트들의 모음이다.

![Memory](https://media.geeksforgeeks.org/wp-content/uploads/memoryLayoutC.jpg)
[출처 : GeeksforGeeks](https://www.geeksforgeeks.org/memory-layout-of-c-program/)

- 프로그램 메모리
- 실행 가능한 기계 코드(Text), 운영체제가 필요로 하는 일부 정보, 런타임 스택(Stack), 동적 할당 메모리 블록(Heap)이 포함된다.

- Range of Virtual Addresses
- 유효한 가상 주소 공간 같은 경우 64비트 머신에서 최대 $2^{64}$ 이다.
	- 하지만 물리적 메모리의 한계로 인해 실제로는 48비트만 사용하고, 나머지 비트의 경우 0으로 설정한다.
	- 즉, $2^{48}$, 64 테라바이트 범위의 바이트를 지정할 수 있다.
	- 프로세스 하나가 이 정도를 지정할 수 있다는 것이기 때문에, 일반적인 프로그램은 보통 메가바이트, 혹은 기가바이트 단위로 접근한다.
- 운영체제는 가상 주소 공간을 관리하고, 가상 주소를 실제 프로세서 메모리의 물리 주소값으로 변환한다.

<br>

---

## 3.2.2 Code Examples

- 아래 코드를 작성했다고 가정하자.

```c
// mstore.c
long mult2(long, long);

void multstore(long x, long y, long *dest){
	long t = mult2(x, y);
	*dest = t;
}
```

### Assembly Code

```shell
linux> gcc -Og -S mstore.c
```

- 어셈블리 코드를 보기위해 -S 옵션을 사용한다.

- 생성된 어셈블리 소스 코드 mstore.s

```assembly
multstore:
	pushq  %rbx
	movq   %rdx, %rbx
	call   mult2
	movq   %rax, (%rbx)
	popq   %rbx
	ret
```

- 각 줄 당 하나의 기계 명령과 일치한다.
	- Ex) pushq : %rbx 레지스터의 내용을 프로그램 스택에 저장하라.

- 생성된 오브젝트 코드 mstore.o
- 아래와 같은 이진 코드가 들어 있다.

```
53 48 89 d3 e8 00 00 00 00 48 89 03 5b c3
```

- 기계가 실행하는 프로그램은 단순히 일련의 바이트로 인코딩된 명령어의 시퀀스이다. 기계는 이 명령어가 생성된 소스 코드에 대한 정보가 거의 없다.


### Assembly Code using Disassembler

- 디스어셈블러(Disassembler)
- 머신 코드 파일의 내용을 확인하기 위해 디스어셈블러 프로그램이 유용하다.
- 디스어셈블러는 머신 코드로부터 어셈블리 코드와 유사한 형식을 생성한다.
- Linux 시스템에서는 `objdump` 프로그램이 -d 플래그를 사용해 이 역할을 수행할 수 있다.

```bash
linux> objdump -d mstore.o
```

- **어셈블리 코드**:
	- 디스어셈블러를 통해 변환된 출력 예시:
```text
// Disassembly of function sum in binary file mstore.o
0000000000000000 <multstore>:
// Offset Bytes              // Equivalent assembly language
       0: 53                    push %rbx
       1: 48 89 d3              mov %rdx,%rbx
	   4: e8 00 00 00 00        callq 9 <multstore+0x9>
	   9: 48 89 03              mov %rax,(%rbx)
	   c: 5b                    pop %rbx
	   d: c3                    retq
```

- 각 기계 명령은 특정 바이트 시퀀스에 매핑된다. (생성된 오브젝트 코드의 나열을 1 $\sim$ 5까지의 바이트 그룹으로 나누어 표현되어 있다.)
- 각 그룹은 하나의 명령어이다.

- **기계 코드 및 어셈블리 코드의 특징**:
	- x86-64 명령은 1바이트 $\sim$ 15바이트까지 길이가 다양하다.
		- 자주 사용되고, 피연산자가 적은 명령어일수록 바이트 길이가 적게 설계되었다.
	- 특정 바이트 값은 유일한 기계 명령으로 해석될 수 있다. (예: `pushq %rbx`는 바이트 값 53으로 시작하는 고유한 명령어이다.)
	- 디스어셈블러는 소스 코드를 필요로 하지 않으며, 기계 코드 파일의 바이트 시퀀스만으로 어셈블리 코드를 결정한다.
	- gcc에 의해 생성된 어셈블리 코드와는 다르게, 디스어셈블러는 일부 접미사를 생략하거나 첨가할 수 있다.
		- 예: 디스어셈블러로 변환한 코드에서 ret 명령어에 q를 붙이는 경우
		- 접미사는 크기 식별자로, 대부분 생략가능하다.

### Executable Code

```c
#include <stdio.h>
void multstore(long, long, long *);

int main() {
	long d;
	multstore(2, 3, &d);
	printf("2 * 3 --> %ld\n", d);
	return 0;
}

long mult2(long a, long b) {
	long s = a * b;
	return s;
}
```

- **링커(Linker)의 역할**:
    - 실행 가능한 코드 생성 위해 객체 코드 파일 세트에서 링커 실행 필요.
    - 링커는 반드시 `main` 함수를 포함한 파일이 필요.
- **예시 코드 (`main.c`)**:
    - `multstore`라는 함수를 통해 두 숫자를 곱하고 결과를 출력하는 프로그램.
    - `long d;`를 통해 결과를 저장할 변수를 선언 후, `multstore(2, 3, &d);` 호출.
    - 결과는 `printf`를 통해 출력됨.

```shell
linux> gcc -Og -o prog main.c mstore.c
```

- `gcc`를 사용해 `main.c`와 `mstore.c`를 컴파일하여 실행 가능한 프로그램 `prog` 생성.
- 결과 파일의 크기는 8,655바이트. 
	- 제공된 절차의 기계 코드와 운영 체제 상호작용을 위한 코드가 포함됨.

### Disassemble Executable Code

```shell
linux> objdump -d prog
```

```code
// Disassembly of function sum in binary file prog
0000000000400540 <multstore>:
	400540: 53                 push %rbx
	400541: 48 89 d3           mov %rdx,%rbx
	400544: e8 42 00 00 00     callq 40058b <mult2>
	400549: 48 89 03           mov %rax,(%rbx)
	40054c: 5b                 pop %rbx
	40054d: c3                 retq
	40054e: 90                 nop
	40054f: 90                 nop
```

- Executable Code와 Object Code 디스어셈블리 결과는 거의 동일하나, 링크에 의해 주소가 변경됨.

- **링커의 역할**:
	- 링커는 함수 호출과 해당 함수의 실행 가능한 코드 위치를 일치시킴.
	- 호출 명령어(`callq`)는 `mult2` 함수의 위치를 참조하도록 수정됨.
- **추가된 코드**:
    - 코드의 마지막에 두 개의 추가 코드 라인이 삽입됨(8-9번째 라인).
        - 이 명령어들은 반환 명령어(7번째 라인) 이후에 위치하여 프로그램에 영향을 미치지 않음.
        - 이 추가 코드는 메모리 성능을 높이기 위해 함수 크기를 16바이트로 확장하는 목적으로 삽입됨.

<br>

---

## 3.2.3 Notes on Formatting

gcc 컴파일러가 생성하는 어셈블리 코드는 사람이 이해하기 어려울 수 있다. 불필요한 정보가 포함되어 있거나 프로그램의 작동 방식이나 목적에 대한 설명이 부족하기 때문이다.

예를 들어, 다음과 같은 명령을 실행하면:
```shell
linux> gcc -Og -S mstore.c
```

```assembly mstore.s
		.file "010-mstore.c"
		.text
		.globl multstore
		.type multstore, @function
multstore:
							 // x는 %rdi, y는 %rsi, dest는 %rdx에 저장됨
		pushq %rbx           // %rbx 저장
		movq %rdx, %rbx      // dest를 %rbx에 복사
		call mult2           // mult2(x, y) 호출
		movq %rax, (%rbx)    // *dest에 결과를 저장
		popq %rbx            // %rbx 복원
		ret                  // 반환
		.size multstore, .-multstore
		.ident "GCC: (Ubuntu 4.8.1-2ubuntu1~12.04) 4.8.1"
		.section       .note.GNU-stack,"", @progbits
```

- ‘.’로 시작하는 모든 줄은 어셈블러와 링커를 위한 지시문으로, 일반적으로 무시해도 된다. 
- 위 text에서는 주석을 추가해놓았으나, 실제로는 주석이 섞여있지 않은 형태이다.
- 즉, 각 명령어가 무엇을 하는지, 원본 C 코드와 어떤 관련이 있는지에 대한 설명은 없다.



### cf) ATT 형식의 어셈블리 코드

- 해당 도서에서는 ATT 형식의 어셈블리 코드를 제시한다.
- 벨 연구소의 운영을 담당했던 AT&T 회사의 이름에서 착안해 지어졌고, gcc, objdump 등의 기본 포맷이다.
- Microsoft, Intel에서 제공하는 어셈블리 코드는 Intel 형식으로 표시된다.

```shell
linux> gcc -Og -S -masm=intel mstore.c
```

```assembly Intel format
multstore:
	push    rbx
	mov     rdx, rbx
	call    mult2
	mov     QWORD PTR (rbx), rax
	pop     rbx
	ret
```

- ATT 형식과 비교한 **Intel 형식**
1. 크기 지정 접미사(q)를 생략한다.
2. 레지스터 이름 앞의 '%' 문자를 생략한다.
3. 메모리 위치를 설명하는 방식이 다르다. 
4. 여러 개의 피연산자를 가진 명령어는 역순으로 나열된다.
	- 두 형식간 전환시 매우 혼란스러울 수 있다.