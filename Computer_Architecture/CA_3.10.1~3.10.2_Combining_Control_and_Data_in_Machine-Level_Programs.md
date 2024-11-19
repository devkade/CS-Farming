# 3.10 Combining Control and Data in Machine-Level Programs

### Table of Contents
1. [Understanding Pointers](./CA_3.10.1~3.10.2_Combining_Control_and_Data_in_Machine-Level_Programs.md#3101-understanding-pointers)
2. [Life in the Real World: Using the GDB Debugger](./CA_3.10.1~3.10.2_Combining_Control_and_Data_in_Machine-Level_Programs.md#3102-life-in-the-real-world-using-the-gdb-debugger)

<br>

# 3.10.0 Overview
- Data와 Control이 상호간에 어떻게 동작하는지에 대한 Chapter.

### What do we study
1. Pointer (w. symbolic debugger GDB)
2. Buffer Overflow
3. Stack Storage

<br>

# 3.10.1 Understanding Pointers
- pointer를 통해, 서로 다른 Data structures 내의 요소들을 일정한 방식으로 참조할 수 있음.

<br>

## i. Principles of pointer (1) Associated type
- 모든 pointer는 *associated type*이 존재함.
    - pointer가 가리키고 있는 객체의 type이 무엇인지 나타냄.
    ```c
    int *ip;
    char **cpp;
    ```

- `void *`는 `generic pointer`를 표현함
- `malloc` 함수는 generic pointer를 반환하는데, 그 generic pointer는 assignment operation의 명시적 혹은 암시적 casting에 의해 `typed pointer`로 바뀜.

<br>

## ii. Principles of Pointer (2) Value
- 모든 pointer는 Value를 가짐.
    - 그 값은 designated type의 어떤 객체의 주소값임.
    - designated type $\approx$ associated type

- `NULL` (0) 값은 pointer가 아무곳도 가리키지 않음을 나타냄.

<br>

## iii. Principles of Pointer (3) `&`
- Pointer는 `&` operator에 의해 생성됨.

- `&` operator는 *lvalue*로 구분되는 모든 C expression에 적용될 수 있음.
    - *lvalue* = assignment에서 왼쪽 편에 나타날 수 있는 표현
- e.g). `leaq` instruction

<br>

## iv. Principles of Pointer (4) `*`
- Pointer는 `*` operator를 통해 dereference(역참조)됨.

- 결과는 pointer와 연관된 type의 value임. ((1)번의.)

<br>

## v. Principles of Pointer (5) Array
- Array와 Pointer는 관계가 깊음.

- Array의 이름이 pointer 변수인 것처럼 참조될 수 있음.
    - Array referencing == Pointer arithmetic + Dereferencing
    - `a[3] == *(a+3)`
        - 이 떄, `a+3`은 Object size에 따라 Offset의 값을 Scaling 해야함.
        - pointer a의 값을 *p*, pointer a와 관련된 data type의 크기를 **L**이라 할 때,<br> a+3 == *p* + **L** * 3

<br>

## vi. Principles of Pointer (6) Casting
- 한 type의 pointer로부터 다른 type의 pointer로 Cast(형 변환)하면, 그것의 type만 바뀌고 Value는 바귀지 않음.
    - (5)에서의 pointer arithmetic의 Scaling을 바꾸기 위해 Casting이 사용됨.

- e.g). `char *` type의 pointer p. *p*를 value로 가지는 상황.
    - (int *) p+7 == *p* + 28
    - (int *) (p+7) == *p* + 7

<br>

## vii. Principles of Pointer (7) Function pointer
- Pointer는 함수도 가리킬 수 있음.

- 코드에 대한 참조를 저장하고 전달하는 기능을 함.
```c
int fun(int x, int *p);

int (*fp)(int, int *);  // int와 int *를 argument로 갖고, int를 반환하는 함수를 가리킴.
fp = fun;

int y = 1;
int result = fp(3, &y);
```
- Function pointer의 값은, 가리켜지는 함수의 machine-code representation에서 첫 번째 Instruction의 주소값임.
    - 예시에서는 함수 fun의 첫 번째 Instruction의 주소값.

- 주의. `int *fp(int, int*);` == `(int *) fp(int, int *)`
    - `*fp`에 소괄호를 쓰지 않으면, `int`와 `int*` 를 argument로 갖고, `int*`를 반환하는 함수 `fp`의 선언이 됨.

<br>

<br>

# 3.10.2 Life in the Real World: Using the GDB Debugger
- GNU debugger인 **GBD**는, machine-level programs의 run-time 평가와 분석을 지원하는 많은 것들을 제공함.

- **GDB**를 사용하면, 실행을 considerable control하면서 Program이 동작하는 것을 보며 공부할 수 있음. (??)

- Figure 3.39의 **GDB** commands를 통해 machine-level x86-64 programs의 동작을 수행할 수 있음.
    - program의 disassembled version을 얻을 수 있는 **OBJDUMP**를 처음 실행하는데 도움됨.

<br>

```shell
linux> gdb prog
```
- 예제들은 file *prog*위에서의 **GDB** 실행에 기반을 둠. 위의 command line으로 **GDB**를 실행.

- 일반적인 방식(Scheme)은 프로그램에서 관심있는 포인트의 근처에 breakpoints를 설정하는 것.
    - 함수의 entry 바로 다음이나, program address에 설정될 수 있음.
- 프로그램 실행 중 breakpoint를 만나면, 프로그램이 중단되고 사용자에게 제어권(control)을 넘김.
    - 이를 통해 해당 시점의 Register와 Memory 위치를 검사할 수 있음.

- CLI인 **GDB**보다 GUI도 제공하는 **DDD**가 있음.