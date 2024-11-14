# 3.5. Arithmetic and Logical Operation

### Table of Contents

1. [Load Effective Address](./CA_3.5_Arithmetic_and_Logical_Operation.md#351-load-effective-address)
2. [Unary and Binary Operations](./CA_3.5_Arithmetic_and_Logical_Operation.md#352-unary-and-binary-operations)
3. [Shift Operations](./CA_3.5_Arithmetic_and_Logical_Operation.md#353-shift-operations)
4. [Discussion](./CA_3.5_Arithmetic_and_Logical_Operation.md#354-discussion)
5. [Special Arithmetic Operations](./CA_3.5_Arithmetic_and_Logical_Operation.md#355-special-arithmetic-operations)

---

# 3.5.0. Overview

## a. What is x86-64?

### i. x86

- also known as **80x86** or the **8086 family**
- 초기에 *Intel 8086* which is 16-bits microprocessor(=CPU)와 여기에 8-bit external bus가 더해진 변종 *Intel 8088*을 기반으로, Intel에 의해 개발된 CISC(Complex Instruction Set Computer) ISA(Instruction Set Architectures)임.

### ii. x86-64

- also known as **x64**, **x86_64**, **AMD64, and Intel 64**
- a 64-bit version of the x86 ISA
- 64-bit mode를 통한 “New paging mode”로, 32-bit(IA-32)보다 더욱 큰 Virtual Memory와 Physical Memory를 지원할 수 있었음.

<br>

- Cf. IA-32: the 32-bits version of the x86 ISA, designed by Intel

<br>

## b. Operations are given as Instruction Classes

- 대부분의 Operations는 Instruction Class로 주어져서, 다른 Size의 Operand를 갖고 변종의 Operation을 가질 수 있게함. (Cf. Only leaq만 다른 Size의 변종이 없음)
    - e.g). ADD
        1. `addb` : add bytes
        2. `addw` : add words
        3. `addl` : add double words
        4. `addq` : add quad words

## c. Four groups of the Operations

1. Load effective Address
2. Unary : 단항 연산자
3. Binary : 이항 연산자
4. Shifts

<br>

# 3.5.1. Load Effective Address

## a. `leaq` : the *load effective address* instruction

- Instruction Format : `leaq S, D`
- Result : `D ← &S`
- `movq`의 변종임
- Memory → Register로 Data를 읽는 Instruction의 Form을 갖고 있지만, 사실 Memory를 참조하지는 않음.
- 첫번째 Operand인 S가 Memory Reference로 나타지만, Designated Location의 Data를 Read하는 것은 아니고, 해당 Destination의 주소값만 복사함.(=`&S`)
- 위의 경우처럼 Pointer로써 사용하기도 하지만, 보통의 Arithmetic Operations를 compact하게 표현하기 위해서도 사용됨.
    - e.g). if register `%rdx`가 value x를 갖고 있을 때, `leaq 7(%rdx, %rdx, 4), %rax` 를 통해 register `%rax`에 `5x+7`이라는 값을 할당할 수 있음.
    - `7(%a, %b, I)` : `$Imm(r_b, r_i, s)$` 의 구조로, `$M[Imm + R[r_b] + R[r_i] * s]$`
    → `M[7 + x + x * 4]` = M[5x + 7]
    → `leaq M[5x + 7], %rax` : Register `%rax`에 `M[5x + 7]`의 주소값(Address), 즉 5x + 7을 할당함.
    - Compiler는 effective address computation과 관련없는 `leaq`의 영리한 사용법(위 예시와 같은)을 찾아내기도 함.

<img src="https://github.com/user-attachments/assets/4d7daa87-eb6f-45fa-89f1-818ec084e61d" title="Figure 3.10" alter="Figure 3.10" width=550>

- Notation
    - $>>_A$ : Arithmetic right shift
    - $>>_L$ : Logical right shift
- cf. SAL == SHL: 두 Instruction의 opcode조차 같기 때문에, CPU도 둘을 구분하지 못함.

## b. The Use of `leaq` in C compiled code

```c
long scale (long x, long y, long z) {
	long t = x + 4 * y + 12 * z;
	return t;
}
```

```assembly
# long scale(long x, long y, long z)
# x in %rdi, y in %rsi, z in %rdx
# -> x,y,z는 주소값이 아닌 실제 Value

scale:
	leaq (%rdi, %rsi, 4), %rax  # x + 4*y
	leaq (%rdx, %rdx, 2), %rdx  # z + 2*z = 3*z
	leaq (%rax, %rdx, 4), %rax  # (x+4*y) + 4*(3*z) = x + 4*y + 12*z
	ret
```

- Interpretation
    - `(%rdi, %rsi, 4)` = `M[R[%rdi] + R[%rsi] * 4]` = `M[x + y*4]`
    → `leaq (%rdi, %rsi, 4), %rax` : Register `%rax`에 x + 4y 값 할당
    - `(%rdx, %rdx, 2)` = `M[z + z * 2]`
    → `leaq (%rdx, %rdx, 2), %rdx` : Register `%rdx`에 3z 값 할당
    - `(%rax, %rdx, 4)` = `M[x+4y + 3z*4]`
    → `leaq (%rax, %rdx, 4), %rax` : Register %rax에 x + 4y + 12z 값 할당
    - `ret` = Return

### → Addition and limited forms of Multiplication의 수행을 `leaq` instruction을 통해 간단하게 구현할 수 있음

<br>

# 3.5.2. Unary and Binary Operations

## a. Unary Operations

- *Source*와 *Destination*으로써 받아지는 Single operand
- Register일수도, Memory Location일수도 있음
- e.g). `incq (%rsp)`
    - Stack 영역의 가장 위에 있는 8-bytes의 Element를 1 증가시킴.
    - C의 increment(`++`) 연산자와, decrement(`--`) 연산자와 유사함.
    - cf. %rsp (Extended Stack Pointer) : 현재 스택의 주소로, 스택의 맨 위쪽 주소를 의미.

## b. Binary Operations

### i. Syntax (1)

- 두 번째 Operand가 *Source*로도, *Destination*으로도 사용될 수 있음.
    - 이 문법은 C의 `x -= y`의 assignment operators와 유사함.
- 첫 번째 Operand로 the Source Operand가 주어지고, 두 번째 Operand로 Destination이 주어지는 경우.
    - noncommutative(비전환적, 비상호적) operations의 입장에선 이상한 format으로 보임.
    - e.g). `subq %rax, %rdx`
        - `%rdx`의 값을 `%rax`의 값으로 빼줌 (= `%rdx`에서 `%rax`를 뺌)
        - `%rdx -= %rax` 이런 느낌이랄까 (여기서도 R[%rdx], R[%rax].
        → "For example, the instruction `subq %rax, %rdx` decrements register `%rdx` by the value in `%rax`.")

### ii. Syntax (2)

- 첫 번째 Operand는 *Immediate Value*(상수), *Register*, or a *Memory Location*이 될 수 있음
- 두 번째 Operand는 *Register*, or a *Memory Location*이 될 수 있음.
- e.g). `mov %a, %b` : `%b`에 `%a`의 값을 할당함
    - 두 Operand는 모두 Memory Location이 될 수 없음.
    - 두 번째 Operand(`%b`)가 Memory Location일 때, the Processor는
        - (1) Memory로부터 Value값을 읽어오고
        - (2) Operation을 수행하고
        - (3) 그 결과값을 다시 Memory에 write back함.

<br>

# 3.5.3. Shift Operations

## a. Syntex
- `SHL k, D`
- 첫 번째 Operand가 *Shift Amount*(=k)를 의미
- 두 번째 Operand가 shift할 *Value*(=D)를 의미
- Destination Operand로 *Register* or a *Memory Location*을 입력 가능함.

### i. The different shift instructions can specify the shift amount either as an immediate value or with the single-byte register %cl

- 첫 번째 Operand인 *Shift Amount*의 값으로, *Immediate Value*(상수) or 1-byte짜리 *Register %cl*을 입력할 수 있음.
    - 특정 Register만 Operand로 입력 가능하다는 점에서 Unusual함.
    - 1-byte 즉, 8-bits의 Shift Amount의 값으로 $2^8 -1 = 255$만큼의 Shift Instruction을 Encoding할 수 있음.
- e.g). x86-64 System에서, $w$-bit인 Data Value(=D)를 연산(Operate)하는 Shift Instruction은 *Register %cl*의 낮은 위치부터 $m$-bits를 *Shift Amount*(=k)로 결정함. (이 때, $w = 2^m$이고, 높은 자리의 bits는 모두 무시됨)
    - 왜 이런 관계가 성립되는가? (Answered by Gemini)
        - **shift amount(=k)의 최댓값** : shift amount(=k)는 0부터 $2^m - 1$까지의 값을 가질 수 있습니다. 예를 들어, $m$이 3이라면 shift amount(=k)는 0부터 7까지의 값을 가질 수 있습니다. ($\because w$만큼 shift하면 Data의 원래의 정보를 잃기 때문.)
        - **데이터 크기와의 관계** : 데이터(=D) 크기가 $w$일 때, 최대 shift amount는 $w-1$이 됩니다. 왜냐하면 데이터를 자기 자신만큼 shift하면 모든 비트가 0 또는 1로 채워지기 때문입니다.
        - $2^m = w$ : 위의 두 가지 조건을 만족하기 위해서는 $2^m = w$라는 관계가 성립해야 합니다. 이를 통해 shift amount가 데이터 크기에 맞게 제한되고, 모든 비트에 대해 유효한 shift가 가능해집니다.
    - e.g). `%cl = 0xFF = b'11111111'` (`sal` = shift left instruction)
        - `salb` : shift by 7 → byte shift는 $w = 8, m = 3$
        - `salw` : shift by 15 → word shift는 $w = 16, m = 4$
        - `sall` : shift by 31 → double word shift는 $w = 32, m = 5$
        - `salq` : shift by 63 → quadword shift는 $w = 64, m = 6$

## b. Two names of Shift Instructions

### i. Left shift instruction

- `sal`, `shl`
    - 오른쪽 bit를 모두 Zero(0)로 채우는, 둘 다 같은 결과값이 나옴.
    - cf. 두 instruction의 opcode는 동일함. 따라서 CPU가 구분하지 못함. 즉, 같은 연산이라는 의미. 

### ii. Right shift instruction

- `sar` = $>>_A$ (arithmetic)
    - Arithmetic shift : Sign bit(=MSB)로 왼쪽 bit를 채움.
- `shr` = $>>_L$ (logical)
    - Logical shift : Zeros(0)로 왼쪽 bit를 채움.

<br>

# 3.5.4. Discussion

- Figure 3.10에서 보여진 대부분의 Instructions는 Data의 부호에 상관없이 사용 가능. (Unsigned or Two’s-complement arithmetic에 둘 다 사용될 수 있음)
- Right Shift Instruction만 Unsigned vs. Signed로 구분되어 각각 Instruction이 존재함 <br>
→ 이러한 특징이, Signed Integer Arithmetic을 구현하기 더 좋은 방법으로 Two’s-complement Arithmetic을 꼽는 이유임.

## a. Figure 3.11 C and Assembly code for arithmetic function

<center><img src="https://github.com/user-attachments/assets/3ddffc56-e5e1-44c8-ae2e-7f9ce6bc8775" width=400 height=400 alt="Figure 3.11" title="Figure 3.11"></center>

- Initially Assignment
    - `R[%rdi]` = x
    - `R[%rsi]` = y
    - `R[%rdx]` = z
- the Destination of the Subtraction = `%rax`<br>
→ `%rax`가 해당 `arith` function의 Return Value가 됨.<br>
cf. `%rax` Extended Accumulator Register : 사칙연산 명령어에서 자동으로 사용, 리턴 레지스터. 시스템콜의 실질적인 번호를 가리키는 레지스터.

- Register `%rax`에 저장되는 Value의 순서가 `3*z`, `48*z`, and `t4`(return value)임.
- 일반적으로 Compiler가 Code를 생성할 때, 그 Code는 (1) Multiple Program Values가 각각 개별적으로 Registers에 저장되고, (2) Registers 사이에서 Program Values를 주고받는 식으로 짜여짐.

<br>

# 3.5.5. Special Arithmetic Operations

- 두 개의 64-bits의 Signed or Unsigned Integers를 곱셈 연산하면, 그 결과를 표현하기 위해 128-bits가 필요함.
- x86-64 Instruction set은 128-bit, 즉 16-bytes의 Numbers와 관련된 연산에 대해 제한적인 Support를 제공함.
    - 2-byte = Word
    - 4-byte = Double Word
    - 8-byte = Quad Word
    - 16-byte = Oct Word (refered by Intel)

## a. Figure 3.12 Special Arithmetic Operations

<center><img src="https://github.com/user-attachments/assets/02e2b204-ab9c-4ac0-b663-45e08671562d" alt="Figure 3.12" title="Figure 3.12" width=550></center>

- 두 개의 64-bit Numbers의 full 128-bit 곱 연산, 나누기 연산을 지원함.

### i. `imulq` Instruction

- has two different forms
1. Two-operand : `IMUL` instruction class
    - `IMUL S, D` : D ← S * D (multiply)
    - 두 개의 64-bit operands를 통해, 하나의 64-bit Product를 generate함.
    - 이를 통해 $*^u_{64}$ and $*^t_{64}$ Operations를 구현함.
        - review).
        $x *^u_{64} y = (x \times y) \space mod \space 2^{64}$<br>
        $\qquad \quad x *^t_{64} y = U2T_{64}((x \times y) \space mod \space 2^{64})$
2. One-operand
    - 두 개의 64-bit values의 곱 결과인 The full 128-bit를 계산하기 위한, 두 종류의 “*one-operand*” multiply instructions로 또 나뉨 (부호에 따라)
    1. `mulq` : for Unsigned
    2. `imulq` : for Signed (Two’s-complement)
    
    - 위 두 Instructions 모두, (`imulq/mulq S`)
        - 한 argument는 Register `%rax`에 있어야하고, 나머지 하나는 *Source Operand*(=`S`)로 주어져야함.
        - 그리고 그 Product의 결과는 Register `%rdx`(high-order 64 bits)와 `%rax`(low-order 64 bits)에 저장됨. → 총 128-bit
        <br><br>
    $\therefore \;$ R[%rdx]:R[%rax] ← S $\times$ R[%rax]

<br>

- 이렇게 `imulq` instruction은 서로 다른 두 개의 format을 갖는데, Assembler는 Operands의 개수를 Count해서 어떤 형식인지 알 수 있음. (일반적인 곱셈:2개 vs. 128-bit의 곱셈:1개)

## b. The following C code that demonstrates the generation of a 128-bit product of two Unsigned 64-bit numbers x and y

```c
#include <inttypes.h>

typedef unsigned __int128 uint128_t;

void store_uprod(uint128_t *dest, uint64_t x, uint64_t y) {
		*dest = x * (uint128_t) y;
}
```

- C Standards의 확장의 일부분으로써 `<inttypes.h>` file에서의 정의를 통해, `x`와 `y`를 64-bit numbers로 명시적으로 선언함.<br>
→ 하지만 이 Standard에서는 128-bit Value를 지원하지 않음.
- 대신, `__int128` 이라는 이름으로 선언된 128-bit Integer에 대한 GCC Compiler의 support에 의존함.<br>
→ `__int128` 을 이용하여 `uint128_t` 라는 data type을 `typedef`함.
- 곱셈의 결과값은 pointer `dest`가 가리키는 16-byte(=128-bit)의 공간에 저장됨.

<br>

```assembly
# The assembly code generated by GCC
# void store_uprod(uint128_t *dest, uint64_t x, uint64_t y)
# dest in %rdi, x in %rsi, y in %rdx

store_uprod:
	movq %rsi, %rax    # Copy x to multiplicand
	mulq %rdx          # Mulutiply by y
	movq %rax, (%rdi)  # Store lower 8 bytes at dest
	movq %rdx, 8(%rdi) # Store upper 8 bytes at dest+8
	ret
```

- 곱셈의 결과값이 두 번의 `movq` instruction에 의해 나뉘어 저장됨.
    - 8-byte(=64-bit)씩 두 번 나눠서 각각. (총 16-byte = 128-bit)
    - cf). Little-endian machine에 맞춰서 Assembly Code가 작성되었기에, High-order bytes를 더 높은 Memory(`8(%rdi)`)에 저장함. Vice versa.

## c. Division and Modulus Operation

### i. Division Operation
- Syntax: `idivl D`
- Single-operand divide instruction : Operand가 하나인 Division (single-operand multiply instruction과 유사함)
    - Dividend(=피제수, 나뉨을 당하는 수)의 값이 Register `%rdx`(high-order)와 `%rax`(low-order)에 미리 64-bit씩 나뉘어 저장돼 있는 상황. 총 128-bit.
    - Divisor(=제수, 나누는 수)는 D, 즉 Instruction Operand로 주어짐.
- 연산의 결과
    - Quotient(몫)는 Register `%rax`에 저장됨
    - Remainder(나머지)는 Register `%rdx`에 저장됨

<br>

- 하지만, 대부분의 64-bit Application에서 Dividend가 128-bit가 아닌, 64-bit로 주어짐.<br>
→ Register `%rax`에 저장돼 있는 Dividend의 값을 64-bit에서 **128-bit로 확장**하는 작업이 필요!
    - 그렇게 하기 위해서는 **high-order 64bits를 저장하는 `%rdx`의 값**을 전부 0으로(Unsigned Arithmetic), 혹은 `%rax`의 Sign bit(MSB)로 전부 바꿔줘야함(Signed Arithmetic).
    - 후자의 작업을 Operand 없이 수행하는 Instruction이, `cqto` (cf. Assembly를 표현하는 표기법으로 Intel 표기법과 AT&T 표기법이 있는데, ATT-format으로는 `cqto`, Intel documentation에서는 `cqt`)

<br>

### ii. Division with x86-64 by C code
```c
void remdiv(long x, long y, long *qp, long *rp) {
    long q = x/y;
    long r = x%y;
    *qp = q;
    *rp = r;
}
```

```assembly
# void remdiv(long x, long y, long *qp, long *rp)
# x in %rdi, y in %rsi, qp in %rdx, rp in %rcx

remdiv:
    movq    %rdx, %r8   # Copy qp → %rdx의 값이 수정될거니까.
                        # 기존 pointer qp의 주소값 잃는것 방지.
    movq    %rdi, %rax  # Move x to lower 8 bytes of dividend
    cqto                # Sign-extend to upper 8 bytes of dividend
    idivq   %rsi        # Divide by y
    movq    %rax, (%r8) # Store quotient at qp
    movq    %rdx, (%rcx)# Store remainder at rp
    ret
```
- Unsigned Division에서는 `divq`를 사용함.<br>
→ 그 전에 `%rdx`를 모두 0으로 Set.