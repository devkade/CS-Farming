# 3.9 Heterogeneous Data Structures

### Table of Contents
1. [Structures](./CA_3.9_Heterogeneous_Data_Structures.md#391-structures)
2. [Unions](./CA_3.9_Heterogeneous_Data_Structures.md#392-unions)
3. [Data Alignment](./CA_3.9_Heterogeneous_Data_Structures.md#393-data-alignment)


<br>

# 3.9.0 Overview
- C는, 2가지 다른 타입의 Objects를 결합하여 새로운 Data types를 만드는 방식을, 두 가지 제공함.

    1. Structures
        - `struct`라는 키워드를 사용하여 선언(declare)함.
        - 하나의 unit에 여러 개의 객체(Objects)들을 모음.

    2. Unions
        - `union`이라는 키워드를 사용하여 선언함.
        - 여러 가지 타입들을 사용하여, 객체를 참조(reference)할 수 있게 함.

### Review
<img src="https://github.com/user-attachments/assets/99076a5e-ee76-4ce3-8a9e-f46fd6570d63" width=550 title="Figure 3.3 Operand forms" alter="Figure 3.3 Operand forms">

- `leaq S, D` == `D ← &S`

<br>

<br>

# 3.9.1 Structures

- C의 `struct` 선언은, 가능하게 다른 타입의 객체들을 하나의 객체로 그룹 지어, 하나의 Data type을 생성함.
- structure의 다른 요소들(Components)은 이름으로 참조됨.

- structure의 구현은 `array`와 유사함.
    1. Structure의 모든 요소들은 Memory에서 **연속적인 영역**에 저장됨.
    2. Structure에 대한 Pointer는 해당 **첫 번째 바이트의 주소값**을 가리킴.<br>
    → 이를 통해 컴파일러는 각 Structure 유형에 대한 정보를, 각 Structure의 메모리 상의 필드의 바이트 오프셋(offset)으로 유지 관리함.<br>
    → 메모리를 참조하는 Instruction에서, 해당 오프셋을 이용하여 Structure의 요소들을 참조함.

```c
// structure declaration
struct rec {
    int i;
    int j;
    int a[2];
    int *p;
};
```
<center><img src="https://github.com/user-attachments/assets/df17b323-e1fc-4be6-b001-0de980383388" width=550 title="the memory field of structure" alter="the memory field of structure"></center>

- 총 4개의 4-byte 크기의 정수형(integer)과, 8-byte 크기의 integer pointer를 요소로 하는 structure를 선언함. $\space \therefore$ 총 24-byte

- (Structure의 Address) + (알맞은 Offset)을 통해 접근하도록, 컴파일러가 코드를 생성함.

## i. Example code of Structure (1) - 요소를 어떻게 참조하나?
- `struct rec *` 타입의 변수 `r`이, Register `%rdi`에 저장돼 있는 상황
```assembly
# Registers: r in %rdi
movl    (%rdi), %eax        # Get r->i      (==(*r).i)
movl    %eax, 4(%rdi)       # Store in r->j (==(*r).j)
```
1. `struct rec`의 객체가 저장돼 있는 메모리의 첫 번째 Byte 주소(e.g. 0x00)를 가리키는 pointer `r`<br>
→ 해당 pointer 객체 `r`이 `%rdi`에 저장돼 있음.<br>
→ 즉, `R[%rdi]` == 0x00

2. `(%rdi)` == `M[R[%rdi]]` == `M[0x00]` == `i`의 값<br>
→ `%eax`에 `i`를 복사함

3. `4(%rdi)` == `M[4 + R[%rdi]]` == `M[0x04]` == `j`의 값<br>
→ `j`의 값이 저장된 곳에, `i`의 값을 복사함.

4. 결과 : `{i, i, a[0], a[1], p}`

<br>

## ii. Example code of Structure (2) - Structure 내부에서 Pointer를 어떻게 생성하나?
- `struct rec *` 타입의 변수 `r`이, Register `%rdi`에 저장돼 있고,
- `long integer` 타입의 변수 `i`가, Register `%rsi`에 저장돼 있는 상황
```assembly
# Registers: r in %rdi, i % rsi
leaq    8(%rdi, %rsi, 4), %rax      # Set %rax to &r->a[i]
```
1. `8(%rdi, %rsi, 4)` == `M[8 + R[%rdi] + R[%rsi] * 4]` == `M[8 + 0x00 + i * 4]` == `M[0x08 + 4 * i]` <br>
→ i에 따라 array a[i]의 Index가 바뀜

2. `leaq M[0x08 + 4 * i], %rax` == `%rax ← 0x08 + 4 * i`

<br>

## iii. Example code of Structure (3) - 최종
```c
r->p = &r->a[r->i + r->j];
```
```assembly
# Registers: r in %rdi
movl    4(%rdi), %eax           # Get r->j
addl    (%rdi), %eax            # Add r->i
cltq                            # Extend to 8 bytes
                                # S = %eax, D = %rax, Sign-Extension
leaq    8(%rdi,%rax,4), %rax    # Compute &r->a[r->i + r->j]
movq    %rax, 16(%rdi)          # Store in r->p
```
1. `%eax`에 `M[4 + R[%rdi]]` 값 할당

2. `%eax`에 `M[0 + R[%rdi]]` 값 더함
3. 8-byte의 Register `%eax`를 16-byte로 Sign-Extension해줌.
4. `%rax`에 `M[8 + R[%rdi] + R[%rax] * 4]`의 주소값, 즉 `r의 첫번째 주소 + 8 + (r->i + r->j) * 4` 할당
5. `M[16 + R[%rdi]]` 즉, `r의 5번째 요소`에 `R[%rax]` 값 할당 ($\because$ `0 + R[%rdi]`가 첫번째 요소)

<br>

## iv. Closing
- 예제 코드와 같이, Structure에서의 다양한 요소(field) 선택은 Compile Time에 완전히 처리됨.
- Machine Code에 Field Declaration(i.e. `int i;`)이나 Field의 이름(i.e. `i`)에 대한 정보는 없음.

<br>

### Cf. Review of C lang
```c
// Example code of Constructor & reference

struct rect {               // rect라는 이름의 structure type
    long llx;               // X coordinate(좌표) of lower-left corner
    long lly;               // Y coordinate of lower-left corner
    unsigned color;         // Coding of color
    unsigned long width;    // Width (in pixels)
    unsigned long height;   // Height (in pixels)
};

struct rect r;
r.llx = r.lly = 0;
r.color = 0xFF00FF;
r.width = 10;
r.height = 20;
// struct rect r = {0, 0, 0xFF00FF, 10, 20};
// Copying하는 방식이 아닌, Pointers를 Pass하는 방식

long area(struct rect *rp) {
    return (*rp).width * (*rp).height;  // *rp.width (X)
                                        // *(rp.width) (X)
                                        // rp->width (O)
}

void rotate_left(struct rect *rp) {
    /* Exchange width and height */
    long t = rp->height; //temp
    rp->height = rp->width;
    rp->width = t;

    /* Shift to new lower-left corner */
    rp->llx -= t; // 왼쪽(반시계방향)으로 돌렸기 때문
}
```

<br>

<br>

# 3.9.2 Unions
## i. Features and Data Structure of Unions
- Union은 C의 type system을 우회하는 방식을 제공함.<br>
→ 하지만, C type system이 제공하는 안전성을 우회하기 때문에, 심각한 Bugs로 이어질 수 있음.

- 단일 Object(아마 Memory상 객체를 의미하는 듯)를 여러 types(Union의 Element)에 따라 참조할 수 있음.
- Structure과 선언 방식은 유사하지만, Memory Allocation에서의 차이가 있음.
    - Structure : 각 Element Field마다, 서로 다른 Memory Block을 참조함.
    - Union : 모든 Element Field가, 하나의 Memory Block을 참조하는데, 그 블록의 크기는 요소 중 크기가 가장 큰 것의 크기 만큼임.

```c
struct S3 {
    char c;
    int i[2];
    double v;
}

union U3 {
    char 3;
    int i[2];
    double v;
}
```
When compiled on an x86-64 Linux machine,<br>
각 Fields의 Offset과, 총 크기 Size.
|Type|c|i|v|Size|
|---|---|---|---|---|
|S3|0|4|16|24|
|U3|0|0|0|8|

- `union U3 *` 타입의 pointer `p`에 대하여, `p->c`, `p->i[0]`, `p->v`는 모두 Data Structure의 시작부분을 참조함.<br>
→ 따라서 Offset이 모두 0임.

- `union U3`의 모든 fields 중 size가 가장 큰 `int i[2]`와 `double v`의 크기인 8-byte가 해당 Data Structure의 총 크기.

<br>

## ii. Union의 활용 (1) 메모리 절감
- 하나의 Application이 한 Data Structure 내부에 있는, 서로 다른 두 Fields를 사용하는 것은 Mutually Exclusive(상호 배타적)하다라는 것을 아는 상황에서.
- 해당 두 Fields를, `Sturcture` 대신에, `Union`을 통해 선언한다면 Total Space Allocated를 줄일 수 있음.

### a. Implement Binary tree data structure with `Structure`
- 각 leaf node에 두 개의 `double` type의 data values가 있고, 각 `internal node`에 data가 없는 두 children에 대한 pointers가 있는, Binary Tree 형태의 구조.

```c
struct node_s {
    struct node_s *left;
    struct node_s *right;
    double data[2];
}
```
- 모든 노드는 32-byte를 필요로 함. (= 8 + 8 + 8*2)

- 이 때, 각 node 타입마다 절반의 bytes가 낭비가 됨. (left, right)

<br>

### b. Implement Binary tree data structure with `Union`
```c
union node_u {
    struct {
        union node_u *left;
        union node_u *right;
    } internal; // structure의 이름이 "internal"
    double data[2];
}
```
- 모든 노드는 16-byte만큼만 필요로 함. <br>
→ 가장 크기가 큰 Element인 double data[2]의 크기 16-byte만큼 할당.

- `union node_u *` type의 pointer 변수 `n`이 있다고 가정할 때,
    - leaf node의 데이터를 참조하기 위해선 `n->data[0]`, `n->data[1]`로 참조
    - internal node의 자식을 참조하기 위해선 `n->internal.left`, `n->internal.right`로 참조

### 이렇게 Encoding하면, 주어진 node가 `leaf`인지 `internal node`인지 알 수 없음
- union을 위한 여러 가능한 선택들을 정의하는 `enumerated` type을 사용함.

- 해당 `enumerated` type인 `tag` field와 `union`을 포함하는 하나의 `Structure`를 만듦.

```c
typedef enum { N_LEAF, N_INTERNAL } nodetype_t; //N_LEAF == 0, N_INTERNAL == 1

struct node_t {
    nodetype_t type; // tag field
    union {          // 하나의 union
        struct {
            struct node_t *left;
            struct node_t *right;
        } internal; // structure의 이름이 "internal"
        double data[2];
    } info; // 마찬가지로 union의 이름이 "info"
}
```
- 위 Data structure은 총 24 bytes만큼 할당됨.
    - 4 for `type`
    - 8 each for `info.internal.left`,`info.internal.right` or `info.data` (둘 중에 더 큰 값)

- 이 때, `type`과 `union` elements 사이에 4 bytes의 padding이 필요함.
    - 따라서, 총 size = 4 + 4(padding) + 8*2 = 24 bytes

- Data Structure의 Field가 많아질수록, 메모리 절감 효과는 커짐.

<br>

## iii. Union의 활용 (2) 다양한 Data type의 Bit patterns Access
- `double` type의 `d`를, `unsigned long` type의 `u`로 형 변환한다고 가정.
```c
unsigned long u = (unsigned long) d;
```
- `u`의 값은 `d`의 Integer Representation(숫자값)으로 됨. (단, d != 0.0)

- `u`의 Bit representation은 `d`의 형태와 매우 다를 것임.
    - e.g). if `d` == 4.0 ($0\space 00...01\space 000...010_2$), `u` == 4.0 ($00...100_2$) (둘 다 64 bits)

<br>

- 아래는 `double` type을 `unsigned long` type의 value로 만드는 코드

```c
unsigned long double2bits(double d) {
    union {
        double d;
        unsigned long u;
    } temp;
    temp.d = d;
    return temp.u;
}
```
- argument의 data type인 `d`와, 목표 data type인 `u`를 하나의 `union`에서 정의

- argument로 받아진 `d`로 `temp.d`의 값을 할당하고, Access할 때에는 `temp.u`를 통해 접근

- 이를 통해 두 value `d`와 `u`는 같은 Bit representation을 갖게 됨.<br>
→ `u`의 숫자값은, `d`가 0.0이 아닌 이상, `d`의 숫자값과 아무런 연관이 없음.

<br>

## iv. Union의 활용 (3) Combining data types of different sizes
- 서로 다른 크기의 Data types를 결합하는데에 `union`을 쓸 때, byte-ordering이 중요해짐
    - review). Little-endian vs. Big-endian
        - Little-endian : 낮은 주소에 데이터의 low-order(낮은 비트, LSB)부터 저장.
        - Big-endian : 낮은 주소에 데이터의 high-order(높은 비트, MSB)부터 저장.
        <img src="https://github.com/user-attachments/assets/c418fec8-0147-4a9e-8e58-350ed4fee0ba" width=300 title="little-endian vs. big-endian" alter="little-endian vs. big-endian">

<br>

- 두 4-byte `unsigned` 값의 Bit patterns를 이용하여, 8-byte `double`을 만드는 상황.
```c
double uu2double(unsigned word0, unsigned word1) {
    union {
        double d;       // 8 bytes
        unsigned u[2];  // 4 bytes * 2
    } temp;             // total 8 bytes

    temp.u[0] = word0;
    temp.u[1] = word1;
    return temp.d;
}
```
- x86-64 processor와 같은 little-endian machine에서, `word0`는 `d`의 low-order 4 bytes가 되고, `word1`은 `d`의 high-order 4 bytes가 됨.

<br>

<br>

# 3.9.3 Data Alignment

## i. Alignment Restrictions
- 많은 Computer systems는 primitive data types(기본 데이터 유형)에 허용되는 주소값들이 제한적임.
- 어떤 객체들에 대한 주소값은 어떤 값 K(일반적으로 2, 4, or 8)의 배수이어야 함.

<br>

- 이러한 *alignment restrictions*(정렬 제한)는 processor와 memory system 사이에 interface를 만들어, 하드웨어의 설계를 단순화함.
- 예를 들어, Processor가 항상, 8 bytes를 memory로부터 8의 배수인 주소값을 통해 fetch(가져오다)한다고 가정.
    - 만약 모든 `double`의 주소값이 8의 배수가 되도록 정렬(align)된다고 하면, 그 값은 single memory operation을 통해 읽거나 쓸 수 있음.
    - 그렇지 않다면, 해당 객체가 두 개의 8-byte memory block에 나뉘어 저장되기 때문에, 두 번의 Memory Access를 통해 수행해야 할 것임.

<br>

## ii. On x86-64 hardware
- x86-64 HW는 데이터의 정렬과 관계없이 정확하게 작동하지만, Intel은 Memory system의 성능 향상을 위해 데이터를 정렬하도록 권장함.

- x86-64의 정렬 규칙은, 모든 K bytes 크기의 primitive object(기본 객체)는 K의 배수의 주소값을 가져야 한다는 규칙을 기본으로 함.

<img src="https://github.com/user-attachments/assets/441e55d7-813c-4058-8daa-66023478ea07" width=250>

<br>

- Compiler는, Global data에 대한 원하는 정렬방식을 나타내는 지시문을 Assembly code에 넣음.
    - Global data는 Global variable을 포함하는 넓은 개념으로, (1)전역 데이터 섹션에 저장되는 상수, (2)정적 변수, (3)컴파일러 생성 데이터(가상 테이블) etc.가 포함됨.

<img src="https://github.com/user-attachments/assets/d60d05d8-f9ef-4ab6-859f-699ca7661f27">

- 2번 라인에서 `.align 8`를 사용하며 jump table을 정의함.
- 해당 지시문의 다음에 나오는 Data는 8의 배수인 주소로 시작함.

<br>

## iii. Alignment in Structure
- `Structure`을 포함하는 코드에서, 컴파일러는 각 structure element가 alignment requirement를 만족하도록 field allocation 사이에 gap을 삽입함. (아마 Padding을 말하는듯?)

```c
struct S1 {
    int i;  // 4 bytes
    char c; // 1 bytes
    int j;  // 4 bytes
};
```
### a. Compiler가 최소한으로 9-byte 할당할 때
<img src="https://github.com/user-attachments/assets/39e90ff5-9206-4392-bd5a-de1979e9b313" width=350>

### b. Compiler가 3-byte gap을 삽입할 때
<img src="https://github.com/user-attachments/assets/eda65642-be48-418c-9e6d-fe219df04564" width=350>

- (a)에서는 4-byte alignment requirement를 충족시킬 수 없음.
    - field `i` (offset 0)
    - field `j` (offset 5)

- (b)에서, field `c`와 `j`의 사이에 gap을 넣음으로써, 충족 가능.
- 만약 `struct S1*` type의 pointer 변수 `p`가 4-byte alignment를 따르고, `p`에 할당된 값이 $x_p$라고 한다면.
    - $x_p$의 값은 4의 배수임.
    - `p->i`(address $x_p$)와 `p->j`(address $x_p + 8$)의 값도 4-byte alignment requirement를 만족함.

<br>

- 반대로 아래의 경우, Structure의 끝에 padding을 추가해야 할 수도 있음.
```c
struct S2 {
    int i;
    int j;
    char c;
};
```

- Padding을 추가하지 않아도, field `i`, `j`, `c` 모두 4의 배수인 주소값을 가지지만,
```c
struct S2 d[4];
```
- 위와 같이 `S2` type을 요소로 갖는 배열 d를 선언한다면, 요소들의 주소값은 $x_d,\space x_d + 9,\space x_d + 18,\space x_d + 27$ 와 같이 만족 X. <br>
→ 따라서 Structure의 끝에 3-byte padding 추가.

<br>

<details>
<summary>Cf. A case of mandatory alignment</summary>
<br>
<p>
    대부분의 x86-64 instructions에서는 Data alignment가 프로그램의 동작에 영향을 주지 않으며 성능을 향상시킴.
</p>
<p>
    반면, Intel과 AMD processor의 어떤 모델은 multimedia operations를 구현하는 SSE instructions에 대해서 unaligned data를 사용하면 올바르지 않게 작동함.
</p>
<p>
    해당 Instructions는 16-byte blocks of data에서 동작함. 그렇지 않은 주소값으로 memory access하면 exception 처리로 프로그램이 종료됨.
</p>
<p>
    그 결과, x86-64 processor를 위한 Compiler와 Run-time system은 SSE register에 저장되고 읽혀지는 Data structure가 16-byte alignment를 만족하도록 보장해야 했음.
</p>
<p>
    alloca, malloc, calloc, realloc 등의 memory allocation function에 의해 생성되는 모든 Block의 시작 주소는 16의 배수여야 함. Stack Frame도 마찬가지.
</p>
<p>
    More recent versions of x86-64 processors implement the AVX multimedia instructions. In addition to providing a superset of the SSE instructions, processors supporting AVX also do not have a mandatory alignment requirement.
</p>
</details>