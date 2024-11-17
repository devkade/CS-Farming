---
up: 
related: 
tags: []
aliases: []
cssclasses:
  - dashboard
created: 2024-11-17T17:02
modified: 2024-11-17T21:34
---
# 3.10.4_Thwarting_Buffer_Overflow_Attacks

## Stack Randomization

- Exploit 코드를 시스템에 삽입하기 위해서 공격자는 (코드 + 코드에 대한 포인터)를 공격 문자열의 일부로 침투시켜야 한다.
- 침투 포인터를 생성하기 위해서는 문자열이 어떤 스택 주소에 위치할지 알아야 한다.
	- 역사적으로 동일한 프로그램, 운영체제 버전의 시스템은 유사한 스택 주소를 사용했기에 예측 가능했다.
	- 때문에 이런 스택 포인터를 알아내어 공격하는 방식에 취약했다.

- Security monoculture : 많은 시스템이 동일한 바이러스에 취약한 현상

- 스택 랜덤화 방식(Stack Randomization)
- 스택 랜덤화 방식의 아이디어는 동일한 프로그램이더라도 프로그램이 실행마다 다른 스택 주소에서 실행되는 것이다.
	- 프로그램 시작시 스택에서 0-n 바이트 사이의 무작위 공간을 할당해 구현된다.
	- alloca() 스택 할당 함수를 통해서 공간을 할당하고, 해당 공간을 사용하지는 않는 방식을 통해서 랜덤하게 만든다.
	- 시스템이 자동적으로 랜덤하게 처리하는 방식이 아닌, 프로그래머가 보안을 위해 공간을 임의로 할당해 랜덤성을 높이는 방식이다.
	- 할당 범위 n은 스택 주소에 충분한 변화를 줄 수 있어야 하지만, 너무 많은 공간을 낭비해서는 안 된다. (범위 조절이 중요하다.)

```c
int main() {
    long local;
    printf("local at %p\n", &local);
    return 0;
}
```

- 해당 코드는 main 함수 내의 local 변수의 주소를 출력한다.
- 32비트 리눅스 머신에서 10,000 번 실행했을 때 주소 범위 0xff7fc59c-0xffffd09c 로, $2^{23}$ 범위
- 64비트 리눅스 머신에서 10,000 번 실행했을 때 주소 범위 0x7fff0001b698-0x7ffffffaa4a8, $2^{32}$ 범위

- cf) Worms and Viruses
	- 컴퓨터 사이에서 스스로 퍼지려 시도하는 코드 조각
	- 웜(Worms)은 스스로 실행될 수 있는 프로그램으로, 다른 머신에 완전 작동하는 버전을 전파할 수 있다.
	- 바이러스(Viruses)는 운영체제를 포함한 다른 프로그램에 자신을 추가하는 코드 조각으로, 독립적으로 실행할 수 없다.

### ASLR(Address-Space Layout Randomization)

- 스택 랜덤화가 리눅스 시스템에서 표준 관행이 되었다.
	- 프로그램 코드, 라이브러리 코드, 스택, 전역 변수, 힙 데이터 등 프로그램의 다양한 부분이 서로 다른 메모리 주소에 load 된다.
	- 서로 매우 다른 주소 매핑을 가질 수 있기 때문에 특정 공격의 형태를 저지할 수 있다.
	- 그러나 랜덤화에 대응해 공격자가 무차별 대입을 시도하여 방어를 뚫을 수 있다.

- Exploit 코드 앞 부분에 nop("no op", short for "no operation") 명령어 시퀀스를 포함시킴으로써 프로그램 카운터를 다음 명령어로 증가시키고, 프로그램 카운터가 exploit 코드에 도달할 수 있도록 무차별 대입을 시도한다.

- n = $2^{23}$ 범위의 랜덤화를 $2^{15}$ = 32,768 nop 명령어를 나열하여 뚫을 수 있다.
- n = $2^{32}$ 범위의 경우 $2^{24}$ = 16,777,216 을 나열해야 하므로 보다 더 어렵다.
- 결국, 스택 랜덤화, ASLR 을 통해서 시스템 공격에 대한 노력을 증가시킬 수 있고, 바이러스, 웜이 퍼지는 비율을 크게 줄일 수 있다.
- 하지만, 완전한 안전 장치를 제공하는 것은 아니다.

## Stack Corruption Detection

- 스택이 손상 되었을 때 일을 탐지 하는 방식을 말한다.
- 프로그램은 앞서 echo 함수에서 배열의 경계를 넘어서는 경우가 발생했을 때 새로운 영향을 미치기 전에 일을 탐지하는 시도를 할 수 있다.

- 최근 버전 gcc 는 코드에 stack protector 라는 매커니즘을 통합하여 버퍼 오버런을 탐지한다.
- 로컬 버퍼와 스택 상태의 나머지 사이의 스택 프레임(Padding)에 특별한 canary 값을 저장한다.

![](https://i.imgur.com/kM63BRF.png)

- 레지스터에 상태를 저장하거나 함수의 반환을 하기 전에, 프로그램은 canary 가 다른 함수에 의해 변경되었는지를 확인한다.
- 만약 변경되었다면 프로그램은 오류와 함께 중단된다.
- 버퍼와 연결되어 있기 때문에, 버퍼가 오버플로우가 발생하는 경우에만 값이 변경된다.

- 최신 버전의 gcc는 함수가 스택 오버플로우에 취약한지 판단하고 stack protector 방식을 자동적으로 삽입한다.
- Stack protector 를 적용한 echo 함수를 컴파일할 경우 다음과 같다.

```assembly
echo:
	subq $24, %rsp          # Allocate 24 bytes on stack
	movq %fs:40, %rax       # Retrieve canary
	movq %rax, 8(%rsp)      # Store on stack
	xorl %eax, %eax         # Zero out register
	movq %rsp, %rdi         # Compute buf as %rsp
	call gets               # Call gets
	movq %rsp, %rdi         # Compute buf as %rsp
	call puts               # Call puts
	movq 8(%rsp), %rax      # Retrieve canary
	xorq %fs:40, %rax       # Compare to stored value
	je .L9                  # If =, goto ok
	call __stack_chk_fail   # Stack corrupted!
.L9:                        # ok:
	addq $24, %rsp          # Deallocate stack space
	ret
```

- %fs:40 은 canary 값을 segmented addressing 방식으로 메모리에서 값을 읽는 명령어 인수이다.
	- Segmented addressing : 메모리를 서로 다른 세그먼트(구역, segment)으로 나누어 구역의 주소를 지정하는 방식을 말한다.
	- 특별한 세크먼트에 canary를 저장함으로써 canary는 읽기 전용으로 표시할 수 있다.
		- 이는 곧 공격자가 canary 값을 덮어쓸 수 없다는 것을 말한다.
- puts 함수까지 동작을 마친 후에는 canary 값을 다시 불러와 xor 을 통해 함수 작동 전 canary 값과 비교한다. 두 값이 동일할 경우 0을 출력한다.

- Stack protector 는 버퍼 오버플로우 공격이 프로그램 스택 상태를 손상시키는 것을 효과적으로 방지한다.
- 기본적으로 성능에 비해 적은 패널티를 가지고, gcc는 함수 내에 char 타입의 로컬 버퍼가 있을 때만 삽입하므로 취약성을 줄이면서도 적은 성능 하락을 갖는다.

## Limiting Executable Code Regions

- 공격자가 실행 가능한 코드를 삽입하는 능력을 제거하는 것.
	- 실행 가능한 코드를 저장할 수 있는 메모리 영역을 제한하는 방식
		- 일반적인 프로그램에서는 컴파일러에 의해 생성된 코드를 지닌 메모리 부분만 실행가능해야 한다.

- 이전 x86 아키텍처는 읽기, 및 실행 접근 제어를 1비트 플래그로 병합했다.
	- 읽기 가능한 모든 페이지가 실행 가능하도록 만들었다.
	- 스택은 읽기 및 쓰기,가 가능하도록 유지해야 하므로 스택 바이트는 실행 가능했다.
	- 스택의 어디서든 실행 가능한 코드를 삽입하면 코드를 실행할 수 있는 것.
	- 해당 아키텍처 구조를 유지하며 스택의 실행 불가하도록 만드는 방법들이 고안되었으나, 효율성 측면에서 떨어졌다.

- 최근 AMD는 64비트 프로세서의 메모리 보호 방식에 NX(no-execute)를 도입했다.
	- 읽기 및 실행 접근 모드를 분리한 방식.
	- 스택은 읽기, 쓰기 가능하지만 실행 가능하지 않도록 표시할 수 있다.
	- 페이지가 실행 가능한지 여부 확인은 하드웨어에서 수행되며 패널티도 없다.

## Summary

- Stack Randomization
- Stack Protector
- Limiting Executable Code Regions

세 방식은 버퍼 오버플로우 공격에 대한 취약점을 최소화하는 데 가장 일반적으로 사용되는 매커니즘이다. 성능 저하가 거의 발생하지 않는 특성을 가지고, 각각이 결합되면 더욱 효과적이다.

