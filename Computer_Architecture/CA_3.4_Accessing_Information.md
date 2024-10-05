---
created: 2024-10-06T01:32
modified: 2024-10-06T06:04
---
# 3.4 Accessing Information

<img width="666" alt="image" src="https://github.com/user-attachments/assets/a6d2ce6a-01a0-4ec0-ab5c-4f2e06d5a4b3">

- x86-64 중앙 처리 장치(CPU)는 64비트 값을 저장하는 16개의 범용 레지스터를 포함.
- 레지스터 이름은 모두 `%r`로 시작하며, 역사적 진화를 반영한 여러 명명 규칙을 따름.
	- 원래의 8086은 8개의 16비트 레지스터(`%ax`부터 `%bp`까지)로 구성되어 있었으며, 각각의 특정 용도를 반영한 이름을 가짐.
	- IA32로의 확장에서 이 레지스터들은 32비트로 확장되어 `%eax`부터 `%ebp`로 라벨링 됨.
	- x86-64로의 확장에서 기존의 8개 레지스터는 64비트로 확장되어 `%rax`부터 `%rbp`로 라벨을 붙임.
	- 추가적으로 8개의 새로운 레지스터가 추가되어 `%r8`부터 `%r15`까지 새 명명 규칙이 적용됨.

- 낮은 자릿수 바이트에 저장된 서로 다른 크기의 데이터에 대해 명령어는 작동 가능.
    - 1바이트 연산: 가장 하위(least significant) 바이트에 접근
    - 2바이트 연산: 가장 하위 2바이트에 접근
    - 4바이트 연산: 가장 하위 4바이트에 접근
    - 8바이트 연산: 전체 레지스터(8바이트)에 접근

> [!Question] 왜 가장 하위 바이트부터 접근할까?
> 1. 구현의 용이성 : 바이트 단위의 데이터 접근이 필요할 때, LSB에서 시작하면 구현이 간단하다. 
> 2. 데이터 처리 효율성 : LSB에서 접근할 경우 하드웨어가 데이터를 처리하는 데 있어 효율성을 높여준다. LSB에서 시작하는 접근 방식은 하드웨어 회로의 복잡성을 줄이고, 더욱 빠르게 특정 비트 조작을 수행할 수 있도록 한다.
> 3. 유연성 발휘 : LSB에서부터 접근하면, 다양한 크기의 데이터 유형(1바이트, 2바이트, 4바이트, 8바이트 등)을 다루는 데 유연성을 제공한다. 작은 데이터 크기 작업을 먼저 한 후, 필요한 경우 더 큰 데이터 처리로 확장할 수 있다.
> - 표현은 big endian을 사용하고, 데이터 접근은 least significant byte에서 할 수 있다.

- 레지스터를 대상으로 하는 명령어에서 생성된 바이트 수가 8바이트 미만인 경우 두 가지 관례가 발생:
1. 1 또는 2 바이트 양으로 생성된 경우: 나머지 바이트는 변경되지 않음.
	- char or short type
	- 기존의 데이터를 보존할 수 있다.
	- 데이터가 읽히거나 쓰일 때 예기치 않은 변화를 방지할 수 있다.
	- e.g. 32비트 레지스터에 1바이트 값을 저장하면, 해당 바이트가 저장된 위치를 제외한 나머지 3바이트는 변경되지 않음.
2. 4 바이트 양으로 생성된 경우: 레지스터의 상위 4바이트가 0으로 설정됨.
	- 레지스터 상위 4바이트는 0으로 초기화된다.
	- 프로그램 안정성을 높이기 위함.
	- 특정 연산에서 과거 데이터가 영향을 미치지 않도록 보장
	- e.g. 64비트 레지스터에 32비트 값을 저장하면, 나머지 32비트는 모두 0으로 설정

> 정확한 이유에 대해서는 추가적으로 검색 필요

- 2번 관례는 IA32에서 x86-64로 확장되는 과정에서 채택됨.

<br>

---

## 3.4.1 Operand Specifiers

<img width="650" alt="image" src="https://github.com/user-attachments/assets/99076a5e-ee76-4ce3-8a9e-f46fd6570d63">

- 대부분의 명령어(instruction)들은 연산을 수행하고 목적지에 결과를 저장하는 1개 이상의 연산자를 가지고 있다.
- x86-64 는 위 표의 연산자를 제공한다.
- Souce value 는 1)상수가 되거나 2)레지스터 혹은 3)메모리에서 load한 값이 될 수 있다.

- **피연산자 유형**
    - **Immediate**: 상수 값을 나타냄.
        - AT&T 형식의 어셈블리 코드에서 Immediate는 '$' 뒤에 정수 표현을 표기해 사용됨.
        - 예 : $-577, %0x1F 
        - 다양한 명령어들은 다양한 범위의 immediate 값을 허용함. 
        - 어셈블러가 자동적으로 값을 가장 간결한 방식으로 인코딩함.
    - **Register**: 레지스터의 내용을 나타냄.
        - 64비트, 32비트, 16비트, 8비트용 레지스터의 저위 부분을 포함.
        - 임의의 레지스터 a에 대한 표기 : $r_a$ 
        - 임의의 레지스터 a의 값 : $R[r_a]$
    - **Memory Reference**: 계산된 주소에 따라 특정 메모리 위치에 접근함.
        - Effective Address : 계산된 주소에 따라 결정된 특정 메모리 위치
        - 메모리는 큰 바이트 배열로 간주되기 때문에, $M_b[Addr]$ 형식으로 메모리의 값에 대한 참조를 나타냄.
        - $M_b[Addr]$ : 메모리에 저장된 주소 $Addr$로 시작하는 b 바이트 값에 대한 참조
        - 여기서 'b'는 바이트 수를 나타내며, 일반적으로는 생략됨.

- 다양한 메모리 참조 형식을 허용하는 많은 addressing mode 가 있다.
- 가장 일반적인 형태 $Imm(r_b, r_i, s)$
	- $Imm$ : Immediate offset
	- $r_b$ : Base register
	- $r_i$ : Index register
	- $s$ : scale factor  {1, 2, 4, 8} 의 값
	- Base register, index register 모두 64비트 레지스터여야 한다.
	- Effective address = $Imm + R[r_b] + R[r_i]\times s$  
	- 해당 형식은 배열의 요소를 참조할 때 자주 사용된다.

### Practice Problem 3.1

| Address | Value | Register | Value |     | Operand        | Value |
| ------- | ----- | -------- | ----- | --- | -------------- | ----- |
| 0x100   | 0xFF  | %rax     | 0x100 |     | %rax           | 0x100 |
| 0x104   | 0xAB  | %rcx     | 0x1   |     | 0x104          | 0xAB  |
| 0x108   | 0x13  | %rdx     | 0x3   |     | $0x108         | 0x108 |
| 0x10C   | 0x11  |          |       |     | (%rax)         | 0xFF  |
|         |       |          |       |     | 4(%rax)        | 0xAB  |
|         |       |          |       |     | 9(%rax,%rdx)   | 0x11  |
|         |       |          |       |     | 260(%rcx,%rdx) | 0x13  |
|         |       |          |       |     | 0xFC(,%rcx,4)  | 0xFF  |
|         |       |          |       |     | (%rax,%rdx,4)  | 0x11  |
<br>

---

## 3.4.2 Data Movement Instructions


- **데이터 이동 명령어의 중요성**
- 데이터의 위치에서 다른 위치로 복사하는 명령어는 컴퓨터 시스템에서 가장 많이 사용됨.
- 피연산자 표기의 일반성 덕분에 간단한 명령어가 다양한 가능성을 표현 가능.

<img width="475" alt="image" src="https://github.com/user-attachments/assets/6a655b52-db93-48c9-977f-5d5ce85ca0e3">

- **데이터 이동 명령어 클래스**
- **mov 클래스**: 데이터의 소스 위치에서 목적지 위치로 데이터를 복사하는 명령어.
- **구성**:
    - `movb`: 1 바이트 복사
    - `movw`: 2 바이트 복사
    - `movl`: 4 바이트 복사
    - `movq`: 8 바이트 복사

- **운영 방식**
- Source operand : Immediate 값, 레지스터 또는 메모리에 저장된 데이터를 지정.
- Destination operand :  레지스터나 메모리 주소를 지정.
- x86-64 아키텍처에서는 명령어의 두 피연산자가 모두 메모리 위치를 참조할 수 없음.
	- 즉, 한 번에 두 메모리 위치 간의 값을 직접적으로 이동할 수 없다.
- 두 메모리 위치 간의 데이터 복사는 두 개의 명령어로 이루어짐: 
    1. 소스 값을 레지스터에 로드
    2. 레지스터 값을 목적지에 저장

- Register operand들은 16개 레지스터 중 특정 레지스터에 해당하고 접미사에 따라 지정된 크기에 따라 값을 저장한다.
- 대부분의 경우 `mov` 명령은 지정된 특정 레지스터 바이트 또는 메모리의 위치만 업데이트한다.
- `mov` 명령의 크기는 destination 의 요구사항에 맞춰야 한다.


- **명령어 예시**

```assembly
	movl $0x4050,%eax      Immediate--Register, 4 bytes
	movw %bp,%sp           Register--Register, 2 bytes
	movb (%rdi,%rcx),%al   Memory--Register, 1 byte
	movb $-17,(%esp)       Immediate--Memory, 1 byte
	movq %rax,-12(%rbp)    Register--Memory, 8 bytes
```

- **특별한 명령어**
- `movabsq`: 임의의 64비트 immediate operand를 source operand로 가질 수 있으며 레지스터만 목적지로 가질 수 있음.

### Understanding how data movement changes a destination register

```assembly
	movabsq $0x0011223344556677, %rax   // %rax = 0011223344556677
	movb    $-1, %al                    // %rax = 00112233445566FF
	movw    $-1, %ax                    // %rax = 001122334455FFFF
	movl    $-1, %eax                   // %rax = 00000000FFFFFFFF
	movq    $-1, %rax                   // %rax = FFFFFFFFFFFFFFFF
```

- `mov` 명령은 값을 복사하는 명령어.
- `movabsq` : 64비트 값을 레지스터 rax에 저장
- `movb` : 상수값 -1을 1바이트만큼 복사
- `movw` : 상수값 -1을 2바이트만큼 복사
- `movl` : 상수값 -1을 4바이트만큼 복사
- `movq` : 상수값 -1을 8바이트만큼 복사

### When copying a smaller source value to a larger destination

<img width="651" alt="image" src="https://github.com/user-attachments/assets/7d05d174-aea5-4b37-8db7-caade271a25d">
<img width="731" alt="image" src="https://github.com/user-attachments/assets/dcd3b7a8-d783-49f3-b1d7-fa67595a114a">

- Source : 레지스터, 메모리에 저장되어 있는 값
- Destination : 레지스터
- movz : 남아있는 비트를 0으로 채운다. (Logical)
	- 4바이트 -> 8바이트가 없는데, 이는 그냥 4바이트를 복사하면 64비트 레지스터에서 복사되지 않은 나머지 비트는 0으로 채워지기 때문이다.
- movs : 남아있는 비트를 부호 확장을 통해 채운다. 부호는 source의 MSB를 사용한다. (Arithmatic)
	- 4바이트 -> 8바이트 확장 명령 존재
- 마지막 2자가 크기 지정자로, 첫 번째는 source 크기, 두 번째는 destination 크기를 지정한다.
- Source (1, 2 바이트) -> Destination (4, 8 바이트)
- `cltq` 명령
    - 운영자가 없는 명령으로, 항상 `%eax`를 소스로, `%rax`를 대상으로 사용.
    - 결과적으로 `movslq %eax, %rax`와 같은 효과를 가짐.
    - 보다 간결한 인코딩을 제공함.

### Practice Problem 3.2

- 올바른 접미사를 붙여보시오

| mov_ | %eax, (%rsp)         | movl |
| ---- | -------------------- | ---- |
| mov_ | (%rax), %dx          | movw |
| mov_ | $0xFF, %bl           | movb |
| mov_ | (%rsp, %rdx, 4), %dl | movb |
| mov_ | (%rdx), %rax         | movq |
| mov_ | %dx, (%rax)          | movw |

### Comparing byte movement instructions

```assembly
	movabsq  $0x0011223344556677, %rax   // %rax = 0011223344556677
	movb     $0xAA, %dl                  // %dl = AA
	movb     %dl, %al                    // %rax = 00112233445566AA
	movsbq   %dl, %rax                   // %rax = FFFFFFFFFFFFFFAA
	movzbq   %dl, %rax                   // %rax = 00000000000000AA
```

- rax에 값 저장. 
- rax, al은 하나의 레지스터에 속함.
- al 에 값이 저장되면서 64비트 레지스터 rax 값도 변화
- movsbq : Source의 MSB를 확인하면 A(1010)이므로 1로 채워준다.
- movzbq : 부호 상관없이 0으로 채우기 때문에 1바이트를 제외하고는 0으로 채워준다.

```assembly
movb $0xF, (%ebx)     // %ebx를 주소 레지스터로 사용할 수 없음 / 접미사와 타입이 맞지 않는다.
movl %rax, (%rsp)     // 접미사와 register id가 일치하지 않는다.
movw (%rax),4(%rsp)   // Source, destination이 모두 메모리 참조를 해서는 안 된다.
movb %al,%sl          // %sl 라는 레지스터는 없다.
movl %eax,$0x123      // 목적지로 immediate 값을 지정할 수 없다.
movl %eax,%dx         // Destination의 크기가 접미사와 일치하지 않는다.
movb %si, 8(%rbp)     // 접미사와 레지스터 id가 일치하지 않는다.
```

<br>

---

## 3.4.3 Data Movement Example

```c
long exchange(long *xp, long y){
	long x = *xp;
	*xp = y;
	return x;
}
```

```assembly
// xp in %rdi, y in %rsi
exchange:
	movq   (%rdi), %rax   // Get x at xp. Set as return value.
	movq   %rsi, (%rdi)   // Store y at xp
	ret                   // Return.
```

- %rax는 함수에서 값을 반환하는 데 사용된다.
- 특징
	- C에서 포인터라고 부르는 것은 단순히 주소일 뿐이다. 
		- 포인터를 역참조하는 것은 포인터를 레지스터에 복사한 다음, 레지스터를 메모리 참조에 사용하는 것이다.
	- x와 같은 지역 변수를 메모리 위치에 저장하는 대신 레지스터에 유지하는 경우가 많다.
		- 레지스터에 접근하는 것은 메모리 접근보다 훨씬 빠르기 때문.



### Practice Problem 3.4

```c
src_t  *sp;
dest_t *dp;

// 다음을 수행하기 위한 적절한 data movement instruction을 사용하려 한다.
*dp = (dest_t) *sp  
```

- sp : %rdi 저장
- dp : %rsi 저장
- %rax, %eax, %ax, %al 사용

| src_t         | dest_t        | Instruction                            | Comment                       |
| ------------- | ------------- | -------------------------------------- | ----------------------------- |
| long          | long          | movq (%rdi), %rax<br>movq %rax, (%rsi) | Read 8 bytes<br>Store 8 bytes |
| char          | int           |                                        |                               |
| char          | unsinged      |                                        |                               |
| unsinged char | long          |                                        |                               |
| int           | char          |                                        |                               |
| unsinged      | unsinged char |                                        |                               |
| char          | short         |                                        |                               |

### Practice Problem 3.5

```assembly
void decode1(long *xp, long *yp, long *zp);

// void decode1(long *xp, long *yp, long *zp)
// xp in %rdi, yp in %rsi, zp in %rdx
decode1:
	movq (%rdi), %r8
	movq (%rsi), %rcx
	movq (%rdx), %rax
	movq %r8, (%rsi)
	movq %rcx, (%rdx)
	movq %rax, (%rdi)
	ret
```

- xp, yp, zp 는 각각 %rdi, %rsi, %rdx 레지스터에 저장되어 있다.
- 어셈블리 코드와 동등한 효과를 낼 수 있는 decode1의 C 코드를 작성하라.

<br>

--- 
## 3.4.4 Pushing and Popping Stack Data

- 스택 : 프로시저 호출 처리에서 중요한 역할을 한다.
- 스택은 후입선출(Last-In, First-Out)의 원칙에 따라서 작동한다. 
- 스택은 배열로 구현할 수 있고, 요소는 항상 하나의 쪽에서 삽입, 제거된다.

<img width="673" alt="image" src="https://github.com/user-attachments/assets/d5b943c3-22f6-4725-9e17-beb1f05684be">

- 초기 상태 -> pushq %rax -> popq %rdx 의 과정을 보여준다.
- pop을 하더라도 값은 다른 push 작업에 의해 덮여쓰일 때까지 해당 메모리 위치 0x104에 남아 있는다.
- 일반적으로 스택은 거꾸로 그려, 스택의 "위쪽"이 아래쪽에 표시된다. 즉, 아래로 성장한다.
- x86-64 에서는 스택이 낮은 주소로 증가하여 스택 포인터(%rsp)를 감소시키고 메모리를 저장한다.
- 쿼드워드의 값을 스택에 푸시하는 것
	- 스택 포인터를 8만큼 감소시킨다. (동적 할당)
	- 새로운 스택 최상위 주소에 값을 쓴다. (값 할당)

```assembly
pushq %rbp  // 해당 명령은 아래 코드와 동급이다.

subq $8,%rsp       // 스택 포인터 감소시키기
movq %rbp,(%rsp)   // 값 저장하기
```
