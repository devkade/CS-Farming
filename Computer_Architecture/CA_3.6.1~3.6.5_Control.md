# 3.6 Control

### Table of Contents
1. [Condition Codes](./CA_3.6.1~3.6.5_Control.md#361-condition-codes)
2. [Accessing the Condition Codes](./CA_3.6.1~3.6.5_Control.md#362-accessing-the-condition-codes)
3. [Jump Instructions](./CA_3.6.1~3.6.5_Control.md#363-jump-instructions)
4. [Jump Instruction Encodings](./CA_3.6.1~3.6.5_Control.md#364-jump-instruction-encodings)
5. [Implementing Conditional Branches with Conditional Control](./CA_3.6.1~3.6.5_Control.md#365-implementing-conditional-branches-with-conditional-control)

<br>

# 3.6.0 Overview of Control
- 이전까지는 Straight-line code의 동작 방식만 살펴봄.
- C의 조건문, 반복문, Swtich문의 구조는 Conditional Execution(조건부 실행)을 필요로 하며, 수행되는 Operations의 순서는 Data에 적용된 Test(조건확인)의 결과에 따라 달라짐.
    - Data Values를 Test(조건확인)한 후, Test 결과(True/False)에 따라 Control flow나 Data flow를 변경함.

- C와 Machine code의 Instructions는 둘 다, Sequentially 실행됨. (Program에 나타나는 순서대로)
    -  Instructions가 실행되는 순서는 `jump` instruction에 의해 바뀔 수 있음.
    → Test 결과에 따라, Control 권한이 Program의 다른 부분으로 넘겨질 수 있음.

- Compiler는 C의 이러한 Control Constructs를 low-level mechanism으로 구현하기 위해, Instruction Sequences를 만들어야 함.

<br>

<br>

# 3.6.1 Condition codes

## Condition Code Register

- CPU는, Integer Register에 더하여, a set of single-bit ***condition code* Registers**를 갖고 있음.
    - 가장 최근의 Arithmetic operation or Logical operation의 특징을 표현하는 Register.
    - 이 Registers로 Test하고, 그것을 바탕으로 Conditional Branches를 수행함.

<br>

## 가장 많이 사용되는 Condition Codes
1. CF: Carry flag
    - 가장 최근의 Operation이 MSB로부터 carry out을 만들어 냈는지에 대한 flag. Unsigned Operations에서의 Overflow를 탐지하는데 사용.
2. ZF: Zero flag
    - 가장 최근의 Operation에서 결과로 0이 나왔는지.
3. SF: Sign flag
    - 가장 최근의 Operation에서 음수의 결과가 나왔는지.
4. OF: Overflow flag
    - 가장 최근의 Operation에서 Two's-complement Overflow가 발생했는지.(negative(-) & positive(+) 둘 다)

### Example
- `t = a+b`를 수행하는 ADD Instruction을 수행하는 상황.
    <img src="https://github.com/user-attachments/assets/d9cfbc26-d354-4f32-8e85-b765d700a1dd" alter="Condition codes example" title="Condition codes example" width=550>

<br>

- `leaq` Instruction은 Address Computation에서 사용되기 때문에, 어떤 Condition Code도 수정하지 않음.
- `leaq`를 제외한 Figure 3.10의 **모든 Instructions**는 Condition code를 수정함.
    - `XOR`은 CF와 OF를 0으로 세팅함.
    - Shift Operations는 마지막에 Shifted out된 bit를 CF에 저장하고, OF는 0으로.
    - `INC`와 `DEC` instructions는 OF,ZF를 조절하고, CF는 건들지 않음.

<br>

<center><img src="https://github.com/user-attachments/assets/db9a5833-5fe4-44d3-8e78-545566e0bb37" alter="Figure 3.13" title="Figure 3.13" width=550></center>

- Register의 값을 바꾸지 않으면서, Condition Codes를 Setting하는 두 가지 Instructions. (Figure 3.13의 Instructions) (having 8-, 16-, 32-, and 64-bit forms)
    1. `CMP`
        - 두 Operand의 차이를 보고 Condition Code를 설정함. `SUB`와 같은 방식이지만, Destination을 갱신하지 않는다는 차이.
        - $S_2 - S_1 == 0$이라면 ZF를 Set.
    2. `TEST`
        - `AND`와 같은 방식이지만, 역시 Destination을 갱신하지 않고, Condition Code를 수정함에서 차이가 있음.
        - 보통 두 Operand로 같은 값을 사용하여, 해당 값이 Negative, Zero, or Positive인지 확인함.
        → e.g). `testq %rax, %rax`
        - 혹은 한 Operand는 어떤 Bit를 Test 해야하는지를 나타내는 Mask 역할.

<br>

<br>

# 3.6.2 Accessing the Condition Codes

## Three ways of using Condition Codes
- Condition Codes를 직접 읽지 않고,  사용되는 다른 방식.
## 1. Single Byte를 Condition Codes의 어떤 조합에 따라 0 or 1로 Setting.
<center><img src="https://github.com/user-attachments/assets/ec392ac7-5674-4212-b104-3920fc2aa87f" width=450 alter="Figure 3.14" title="Figure 3.14"></center>

- `SET`이라는 Instruction으로, 고려되는 Condition Codes의 조합의 다름에 따라, Suffix도 달라짐. (e.g. sete, sets, setns, etc.)<br>
→ 여기서의 Suffix는 다른 크기의 Data를 Operand로 하는 의미가 아님. (!= `salb` etc.)
- .1) low-order single-byte Register elements, 혹은 2) single-byte Memory Location을 Destination으로서 가짐. → 해당 Byte를 0 or 1로 Setting.

```assembly
# Example of Compare a and b
comp:
    cmpq    %rsi, %rdi  # Compare a:b
    setl    %al         # Set low-order byte of %eax to 0 or 1
    movzbl  %al, %eax   # Clear rest of %eax (and rest of %rax)
    ret
```
- cf. `movzbl`은 Unsigned 1-byte 정수를 Unsigned 4-byte 정수로 변환하는 명령어  
→ `%al`에 있는 1-byte 정수를, `%eax`에 Zero Extension하여 4-byte 정수로 할당.
    - 추가로, `%rax`의 upper 4-byte도 0으로 clear해줌.

<center><img src="https://github.com/user-attachments/assets/e06e924a-a1f2-4bbe-b1f7-82da1aa86600" width=400></center>

- `%al`은 `%eax`의 low-order byte (%EAX[0:7]=1byte)

<br>

- Figure 3.14에서 Synonym은 동의어를 의미함. Compilers와 Disassembler는 동의어 중, 어떤 이름을 사용할지 임의로 선택함.

### i. $t = a -^t_w b$를 수행할 때, 여러 `SET` Instruction의 수행 결과
|condition|CF|ZF|SF|OF|
|:---|:---:|:---:|:---:|:---:|
|`sete`, when a = b|0|1|0|0|
|`setl`, when no overflow occurs<br>&$\; a -^t_w b < 0$<br> $\therefore a < b$|0|0|1|0|
|`setl`, when no overflow occurs<br>&$\; a -^t_w b \ge 0$<br> $\therefore a \ge b$|0|?|0|0|
|`setl`, when overflow occurs<br>&$\; a -^t_w b > 0$ (negative overflow)<br> $\therefore a < 0 < b$|0?|0|0|1|
|`setl`, when overflow occurs<br>&$\; a -^t_w b < 0$ (positive overflow)<br> $\therefore b < 0 < a$|1?|0|1|1|

- OF와 SF의 `EXCLUSIVE-OR` 연산으로 $a < b$인지 확인할 수 있음.
    - `EXCLUSIVE-OR` 연산값이 1이면 $a<b,\quad$ 0이면 $a>b$.

### ii. $t = a -^u_w b$를 수행할 때
- CF와 ZF의 조합으로 대소를 비교함.
- $a-b<0$일 때, `CMP` Instruction이 CF를 1로 Set함.

## 2. Program의 다른 부분으로 Jump
- CA_3.6.4에서 다룰 예정.

## 3. 조건부로 Data를 전송
- ??

### C와 달리, Machine Code가 Signed vs. Unsigned를 어떻게 구분하는지는 중요하지 않음.
- 두 경우 모두, 대부분 같은 Instruction으로 같은 동작을 bit-level에서 수행하기 때문.
- 단, Right Shift, Division, and Multiplication은 제외.
- 추가로, Condition Codes의 조합도 서로 다르게.

<br>

<br>

# 3.6.3. Jump Instructions

- 보통의 실행에서는, Instruction은 Listed된 순서대로 이어짐.
- but, `jump` Instruction을 통해, Program에서 다른 위치로 이동할 수 있음.
- `jump` Destination은 `Label`이라는 code를 사용함.

<center><img src="https://github.com/user-attachments/assets/68d1dc46-84d2-4817-a959-ae4e21548037" width="500" title="Figure 3.15" alter="Figure 3.15"></center>

```assembly
    movq    $0, %rax    # Set %rax to 0
    jmp     .L1         # Goto .L1
    movq    (%rax), %rdx # Null pointer dereference(skipped)
.L1:
    popq    %rax        # Jump target
```

- `jmp .L1`을 통해 `movq` Instruction을 스킵하고, `popq` Instruction을 수행함.

- Object-code File을 만들 때, Assembler가 Labeled Instructions(e.g. `.L1`)의 주소값을 결정하고, `jump target`을 `jump` Instruction(e.g. `jmp .L1`)의 일부로 encode함.

### i. `jmp` Instruction
- Unconditional Instruction

- `jmp` Instruction은 두 가지로 나뉨.
    1. direct jump
        - Assembler에 의해 encode되어 Jump Instruction의 일부가 된 jump target으로 이동.
        - target으로 Label을 입력받음.

    2. indirect jump
        - Register or Memory Location으로부터 jump target을 읽어서 이동. 이는 Operand로 입력됨. (e.g. **Operand* )

### ii. Operand specifier `*`
- `jmp *%rax`
    - register `%rax`에 저장돼 있는 Value를 jump target으로 함.

- `jmp *(%rax)`
    - `%rax`에 저장된 Value를 Address로 하여, Memory로부터 jump target을 읽어옴.

### iii. Conditional Jump Instructions
- `jmp`를 제외한 Figure 3.15의 나머지 Instruction은 Conditional.

- 특정 Condition Codes의 조합에 따라, (1) target으로 Jump할지, (2) code sequence에 따라 다음 Instruction을 순차적으로 실행할지가 결정됨.

- Instruction의 Suffix들은 `SET` Instruction(in Figure 3.14 of CA_3.6.2)의 Suffix와 동일함. (e.g. `setl == set less`, `jl == jump less`)

- Conditional Jump Instruction은 Direct밖에 안 됨. (Indirect는 사용불가)

<br>

<br>

# 3.6.4. Jump Instruction Encodings

- 지금까지의 대부분의 Part에서는, Machine Code의 디테일한 포멧을 딱히 신경 안 썼는데.
- 그러나, the Targets of Jump Instructions가 어떻게 Encoding되는지가 이후에 Chapter 7의 Linking에서 중요함.(과연 여기까지 갈 수 있을지)

- 그리고, Disassembler의 결과값을 햏석하는데 도움이 됨.

## Encoding Methods
### i. PC relative
- Assembler와 Linker가 Jump Targets의 적절한 Encode를 만드는데, 여러 Encodings 중, *PC relative*라는 것이 가장 많이 쓰임.
    - (1) Address of the target Instruction과 (2) Address of jump 다음에 바로 나오는 Instruction의 차이를 Encode하는 방식.

    - 해당 Offset은 1, 2, or 4 bytes를 이용하여 Encode할 수 있음.

### ii. By giving an Absolute Address
- 4 btyes의 Absolute Address를 이용하여, direct하게 target을 특정함.

→ 이들 중, Assembler와 Linker는 적절한 방식을 선택하여 Jump Instruction을 Encoding함.

## Example of PC-relative addressing
```assembly
1       movq    %rdi, %rax
2       jmp     .L2
3   .L3:
4       sarq    $rax
5   .L2:
6       testq   %rax, %rax
7       jg      .L3
8       rep; ret
```
```
# disasemmbled version of the .o format generated by the assembler

1   0:  48 89 f8    mov     %rdi, %rax
2   3:  eb 03       jmp     8 <loop+0x8>
3   5:  48 d1 f8    sar     %rax
4   8:  48 85 c0    test    %rax, %rax
5   b:  7f f8       jg      5 <loop+0x5>
6   d:  f3 c3       repz retq
```

- line 2의 `0x8`과 line 5의 `0x5`가 각각의 jump target으로 되어 있음.

- Byte Encodings를 보면, line 2의 `0x03`과 line 5의 `0xf8`으로 Encode되어 있음.
    - `0x03`(offset)과 `0x05`(following address)를 더해, target address인 `0x08`로 jump함.
    - `0xf8`(offset)과 `0x0d`(following address)을 더해, -8 + 13 = 5, 즉 `0x05`로 jump함.

- Program이 실행되면서 PC(Program Counter)에 저장되는 값은, Jump Instruction 자체의 주소값이 아닌, 그 다음에 나오는 Instruction의 주소값이 저장됨.
    - Instruction을 수행하기 이전에, Processor(CPU)가 PC에 그 다음 주소값으로 갱신하기 때문.

<br>

```
# disassembled version of the program after Linking.

1   4004d0: 48 89 f8        mov     %rdi, %rax
2   4004d3: eb 03           jmp     4004d8 <loop+0x8>
3   4004d5: 48 d1 f8        sar     %rax
4   4004d8: 48 85 c0        test    %rax, %rax
5   4004db: 7f f8           jg      4004d5 <loop+0x5>
6   4004dd: f3 c3           repz retq
```
- Linking 이후에, Instruction의 Address들은 바뀌었지만(`0` → `4004d0`), Jump target의 Encode는 바뀌지 않음(`03`, `f8`)

- PC-relative Encoding을 사용하면, 2 bytes만 사용하고 Compactly Encode가 가능하고, Object Code의 수정없이 Memory의 다른 위치로 이동이 가능함.

<br>

<br>

# 3.6.5. Implementing Conditional Branches with Conditional Control

- C에서의 Conditional Expressions and Statements를, Machine Code로 표현하는 대표적인 방법으로는, (1) Conditional and (2) Unconditional Jumps의 조합을 사용하는 것.

<center><img src="https://github.com/user-attachments/assets/a3e62490-9c08-412d-b9de-27bdf999ea73" title="Figure 3.16" alter="Figure 3.16" width=450></center>

- (a)는 두 수의 차이의 절대값을 구하는 함수 in C code. Global variable `lt_cnt`와 `ge_cnt`를 증가시키기도 하는.

- (c)는 GCC가 생성한 Assembly Code.

- (b)는 (c)를 다시 C code로 변환했을 때의 Code로, Assembly에서 사용되는 Unconditional Jump인 `goto`를 사용함.

### cf. "goto code"
> 프로그램을 읽고 디버그하는게 힘들기 때문에, `goto` 구문을 사용하지 않는데,
> Machine code의 **control flow**를 C program으로 해석하기 위해, 이 책에서는 `goto`를 사용함. 이 programming style을 "**goto code**"라고 함

<br>

### (c) Assembly Code의 Control Flow
1. 두 Operand를 비교하여 Condition Codes를 세팅함.
2. $x \ge y$일 때, `.L2`로 jump함.
3. Global variable `ge_cnt`에 1을 더하고, `x-y`를 계산하여 return 값으로 저장하고, return함.

- 반대로, $x < y$일 때는, 이어지는 Instruction인 `addq`연산을 통해 `lt_cnt`를 1 증가시키고, `y-x`를 계산하여 return함.

→ (c)의 Control flow가 (b)와 유사하다는 것을 알 수 있음.

## General form of an if-else statement in C & Assembly
```c
if (test-expr)
    "then-statement"
else
    "else-statement"
```

```assembly
# Control flow를 표현하기 위한 C syntax 사용

    t = test-expr;
    if (!t)
        goto false;
    then-statement
    goto done;
false:
    else-statement
done:
```