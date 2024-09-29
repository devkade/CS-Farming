# 2.2. Integer Representation

### Table of Contents

0. [Overview](./CA_2.2_Integer_Representation.md#220-overview)
1. [Integral Data Types](./CA_2.2_Integer_Representation.md#221-integral-data-types)
2. [Unsigned Encodings](./CA_2.2_Integer_Representation.md#222-unsigned-encodings)
3. [Two’s-Complement Encodings](./CA_2.2_Integer_Representation.md#223-twos-complement-encodings)
4. [Conversions between Signed and Unsigned](./CA_2.2_Integer_Representation.md#224-conversions-between-signed-and-unsigned)
5. [Signed versus Unsigned in C](./CA_2.2_Integer_Representation.md#225-signed-versus-unsigned-in-c)
6. [Expanding the Bit Representation of a Number](./CA_2.2_Integer_Representation.md#226-expanding-the-bit-representation-of-a-number)
7. [Truncating Numbers](./CA_2.2_Integer_Representation.md#227-truncating-numbers)
8. [Advice on Signed versus Unsigned](./CA_2.2_Integer_Representation.md#228-advice-on-signed-versus-unsigned)


# 2.2.0. Overview

## a. Two Ways of Representing Integers

1. Only Non-negative Numbers (unsigned)
2. Negative, Zero, and Positive Numbers

→ 수학적 특성과 Machine-level에서의 Implementation에서 강한 Relation을 가짐

## b. Expand or Shrink an encoded integer to fit a representation with a different length.

- 길이가 다른 표현과 일치하는 인코딩된 정수를 확장하거나 축소함

## c. Terminology for Integer Data and Arithmetic Operations

![Figure 2.8](https://github.com/user-attachments/assets/1ce8590b-871d-4c44-80ba-f4e29d65f919)

- w는 Bit 수
- B2T = Binary-to-Two's Complement
- B2U = Binary-to-Unsigned
- …
- TMin = Two's-complement value's Minimum
- TMax = Two's-complement value's Maximum
- UMax = Maximum Unsigned Value


# 2.2.1. Integral Data Types

## a. Clang은 다양한 Integer의 Data Type을 지원함

![Figure 2.9](https://github.com/user-attachments/assets/87a97b70-a066-4da9-81bb-5a537f1eb13a)

![Figure 2.10](https://github.com/user-attachments/assets/70f4e2a5-a1ef-49c0-bc39-41f3b7367e50)

- 32-bit Program인지, 64-bit Program인지에 따라 표현되는 정수의 유한한 범위 또한 다름
<details>
<summary>Cf.</summary>
<div markdown="1">
<li>32-bit, 64-bit는 CPU 내의 Register가 한 번에 처리할 수 있는 Bit의 수를 의미함.</li>
<li>Q01. 그런데 왜 char, short, int, int32_t, int64_t에서는 차이가 없고, long type에서만 차이가 있나요?</li>
<li>A01. 아래 표와 같이 OS는 각각의 Data Model을 채택하여 사용 중인데, 그 Data Model이 C에서의 Data Type의 Bit Size를 다음과 같이 지정하였기 때문에 차이가 있기도 하고 없기도 합니다.</li>
    
<li>Q02. 그럼 32-bit Program 위에서 크기가 64bit인 int64_t data type은 어떻게 사용하나요? 작동이 되나요?</li>
<li>A02. 됩니다. 32-bit의 Program은 해당 Data를 저장 및 표현하기 위해 2개의 Memory Address를 사용합니다.</li>

<img width="585" alt="image" src="https://github.com/user-attachments/assets/81ff0dbd-c696-47ac-bede-b48c51325a32">
<br>
https://en.wikipedia.org/wiki/64-bit_computing#64-bit_data_models
  
</div>
</details>
<br>

- char, short, long으로 특정되는데, 이들은 Unsigned or Possibly negative 이렇게 두 표현으로도 특정됨
- Compile된 Program이 32-bit인지 64-bit인지에 따라 범위가 결정되는 것은 long밖에 없음
- 양수의 범위에 비해, 음수의 범위가 1 더 많음 (e.g. -128 vs. +127)
  - 음수가 표현되는 방식에 의해 그렇게 됨


# 2.2.2. Unsigned Encodings

- $\overrightarrow{x} = [x_{w-1}, x_{w-2}, \dots, x_0]$  (단, $x_i = 0 \space or \space 1$)
- $B2U_w(\overrightarrow{x}) \doteq \sum^{w-1}_{i=0}x_i2^i$  (Binary-to-Unsigned)
→ 우리가 흔히 아는 2진수 10진수로 변환하는 함수
- $UMax_w \doteq 2^w - 1$  (Unsigned Value Maximum)
<br>

### PRINCIPLE : Uniqueness of Unsigned Encoding

- There is only One Representation of Decimal value N as an Unsigned w-bit Number. (일대일 대응)
→ Function $B2U_w$ is a bijection (bijection = a Function $f$ that goes two ways = 역함수 존재)
→ $U2B_w$ Exists (Unsigned-to-Binary)



# 2.2.3. Two’s-Complement Encodings

- 보통의 Computer에서는 Signed Numbers를 필요로 하기 때문에, 이를 표현하기 위한 방식으로 Two’s-complement form을 사용함(2의 보수)
    - The Most Significant Bit(MSB, 일명 Sign Bit) of the word have Negative weight
    → 가장 왼쪽의 Bit가 양수, 음수의 부호를 결정함
<br>

- $\overrightarrow{x} = [x_{w-1}, x_{w-2}, \dots, x_0]$  (단, $x_i = 0 \space or \space 1$)
- $B2T_w(\overrightarrow{x}) \doteq -x_{w-1}2^{w-1} + \sum_{i=0}^{w-2}x_i2^i$ (Binary-to-Two’s complement)
→ For 4-bit representation, [1001] = -7, [0111] = 7, 둘을 더하면 [0000] = 0
최소값은 [1000] = -8
- $TMin_w \doteq -2^{w-1}$
- $TMax_w \doteq \sum^{w-2}_{i=0}2^i = 2^{w-1}-1$
- The Range of $B2T_w$ = $B2T_w:\{0,1\}^w \rightarrow \{TMin_w, \dots , TMax_w\}$
<br>

### PRINCIPLE : Uniqueness of two’s-complement encoding

- Function $B2T_w$ is a bijection (일대일 대응, 즉 역함수를 가짐)
- $T2B_w$ exists  (Two’s-complement-to-Binary)
<br>

### Practice Problem 2.17

![Practice Problem 2.17](https://github.com/user-attachments/assets/4dc11363-d649-46e0-b8f4-117951bd100a)
<br>
## a. Figure 2.14_Important Numbers

![Figure 2.14](https://github.com/user-attachments/assets/f706ca1f-4465-4702-aecd-6b7ae8f6af82)

w의 단위는 bit

- 앞으로 $UMax_w$, $TMin_w$, and $TMax_w$ 이 세 가지 Special Values는 앞으로 많이 쓰일 것이고, Subscript $w$는 생략함
- 4-bit씩 묶어서 Hexadecimal의 한 자리를 표현함
- Two’s-complement에서 최소값을 갖는 Bit Representation이 0x80..0인 이유는 Binary로 표현했을 때 1000..0000이 최소값을 갖기 때문
    - 반대로 최대값을 갖는 Bit Representation이 0x7F…FF인 이유는 Binary로 표현했을 때, 0111…1111이 최대값을 갖기 때문

### i. The Two’s-complement range is asymmetric: $|TMin| = |TMax| + 1$; that is, there is no positive counterpart to $TMin$

- 이 때문에 Subtle Program bugs로 이어질 수 있음
- Bit Patterns로 표현되는 것들 중 절반은 Negative Numbers, 나머지 절반은 Nonnegative Numbers를 표현함
→ 그런데 0은 Nonnegative Number에 포함되므로, Negative Numbers의 수가 하나 더 많은 것
<br>

### ii. $UMax = 2TMax + 1$

- Unsigned Value의 최대값은 Two’s-complement Value의 두 배가 넘음
<br>

### iii. Representation of Constants -1 and 0

- -1은 $UMax$와 동일한 Bit Representation을 가짐 0xFFFF…FFFF
- 0을 Signed이거나 Unsigned이거나, 모두 0x0000…0000으로 모든 Bit가 0
<br>

## b. A Portability related to Representation of Signed Numbers

- C standards에서는 Signed Number를 Two’s-complement로 표현하라고 요구하지는 않지만, 대부분의 Machine에서는 요구함.
- Portability를 중시하는 개발자는, Figure 2.11에 표시된 Range를 넘어서는 특정 범위의 표현 값을 가정해서는 안 되며, Signed Number의 특정 표현도 가정해서는 안 됨.
→ C Standards가 보장하는 범위 내의 Signed Number는 Two’s-complement로 표현한다고 가정하고, Range를 넘어서는 안 됨.

![Figure 2.11](https://github.com/user-attachments/assets/0ac3166a-e7f5-4da4-870f-5464695ae3f9)

- `<limits.h>` file(in the C library)를 통해 Integer Data Typed의 범위를 동적으로 지정(delimit)하여, Compiler가 동작 중인 Machine이 요구하는 범위로 맞춰줌.
    - `INT_MAX`, `INT_MIN`, `UINT_MAX`를 사용하여 Signed and Unsigned Integers의 범위를 동적으로 정의함.
    → Two’s-complement machine에서의 w-bits의 Integer의 경우, 위 세 상수는 각각 $TMAX_w$, $TMin_w$, and $UMax_w$로 값이 매칭됨.
<br>

### cf. The Java Standard

- Java Standard는 64-bit case에서 보여진 Range와 Representation을 따름. (Figure 2.10)
- Single-byte(1-byte) Data type인 `char`를 `byte`로 부름.
    - 이를 통해 Machine이나 OS에 상관없이, Java Program은 Identically 작동할 수 있음.
<br>

## c. Figure 2.15 Two’s-complement representations of 12,345 and -12,345, and unsigned representation of 53,191

```c
short x = 12345;
short mx = -x;

show_bytes((byte_pointer) &x, sizeof(short));
show_bytes((byte_pointer) &mx, sizeof(short));
```

![Figure 2.15](https://github.com/user-attachments/assets/b74ff390-0824-4dc5-b2b9-f7d09ae862be)

- Weight : Two’s-complement Representation에서 각 Bit가 갖는 가중치
- Bit : 위에서부터 $x_0$ ($[x_{w-1}, x_{w-2}, \dots , x_1, x_0]$)
- Value : $Weight \times Bit$
- 결과
    
    ![Code Result 1](https://github.com/user-attachments/assets/2fc68b3e-3d89-458b-b641-1982747e9dc5)
    
    - 12,345 = [0 0 1 1 0 0 0 0 0 0 1 1 1 0 0 1] = 0x3039
    - -12,345 = [1 1 0 0 1 1 1 1 1 1 0 0 0 1 1 1] = 0xCFC7
<br>

### Cf. More on Fixed-size Integer types
<details>
  <summary>Contents</summary>
  <div markdown="1">
    <h3> a. It is essential that Data types be encoded using representations with specific sizes.</h3>
    <ul>
      <li> Standard Protocol을 이용한 Internet에서 Communicate하기 위해서는 Protocol이 제한한(Specified) 호환 가능한 Data Types로 Program을 짜야함</li>
      <li> e.g). C의 Data types으로 `long`은 호환가능은 하지만, Machine마다 범위가 다름<br>
          		→ 따라서 호환되는 Data Type을 선택했다 하더라도, Portability를 보장할 수는 없음 </li>
    </ul>
    <h3> b. The ISO C99 standard introduces this class of integer types in the file stdint.h</h3>
    <ul>
      <li> Figure 2.3에서 다뤘던 32- and 64-bit versions of fixed-sized integer types에 대해서 국제표준화기구 ISO C99 standard가 <code>stdint.h</code>에 정의해 놓음</li>
      <li><code>stdint.h</code> file에서 <code>intN_t</code>와 <code>uintN_t</code>의 형태로 Integer를 정의해 놓음
        <ul>
          <li>N의 정확한 값은 Implementation dependent하지만, 대부분의 Compiler는 8, 16, 32, 64를 허용함
          </li>
        </ul>
      </li>
    </ul>
    <h3> c. Along with these data types are a set of macros defining the minimum and maximum values for each value of N. These have names of the form INTN_MIN, INTN_MAX, and UINTN_MAX. → 무슨 Macro를 의미하는 건지 모르겠음</h3>
    <li> 정해진 순서에 따라 어떻게 특정한 입력 시퀀스 (문자열을 가리키기도 함)가 출력 시퀀스 (이 또한 문자열을 가리키기도 함)로 매핑되어야 하는지를 정의하는 규칙이나 패턴을 말한다. 하나의 매크로를 특정한 출력 시퀀스로 바로 만들어내는 매핑 과정은 "매크로 확장"이라고 알려져 있다.<br>
    → from <a href="https://ko.wikipedia.org/wiki/%EB%A7%A4%ED%81%AC%EB%A1%9C_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99)">[매크로 (컴퓨터 과학) wikipedia]</a>
    </li>
    <li>
      앞에서 말한 Context의 개념인가??
    </li>
    <h3> <code>
      printf("x = %" PRId32 ", y = %" PRIu64 "\n", x, y);</code></h3>
    <ul>
      <li>변수 x의 data type : int32_t</li>
      <li>변수 y의 data type : uint64_t</li>
      <li>PRId32 : `stdint.h`에 정의된 Macro로, 32-bit Integer를 플랫폼에 맞는 적절한 형식으로 출력하기 위해 사용됨. 사실상 `printf`의 형식 지정자 `%d`와 동일하게 작동함. 플랫폼 간의 이식성(Portability)을 보장하기 위해 표준으로 정의된 매크로임.</li>
      <ul>
        <li><code>%d</code> : Signed Integer data type을 출력할 때 사용되는 <code>printf</code>의 형식 지정자. 32- and 64-bit System에서 일반적으로 <code>int</code> type의 값을 처리하고, 그 값은 C Standard에 따라 4-bytes의 크기를 가짐.</li>
      </ul>
      <li>PRIu64 : <code>uint64_t</code> type의 값을 출력하는 형식 지정자임. <code>PRId32</code>와 유사함.</li>
      <ul>
        <li><code>%llu</code> : Unsigned 64-bit Integer data type을 출력할 때 사용되는 <code>printf</code>의 형식 지정자. e.g). <code>unsigned long long</code> or <code>uint64_t</code></li>
      </ul>
      <li>결과</li>
          <ul>
            <li><code>x = "x의 값", y = "y의 값"</code></li>
          </ul>
                
    
    <h3> d. Using the macros ensures that a correct format string will be generated regardless of how the code is compiled.</h3>
    
      <li> Macro를 사용하면, Code가 어떻게 Compile되던지 간에, 올바른 format의 String이 생성되는 것을 보장할 수 있음</li>
</div>
</details>
  
  
### Cf. Alternative Representation of Signed Numbers
<details>
  <summary>Contents</summary>
  <div markdown="1">
    <h3> i. Ones’ complement (1의 보수)</h3>
    <li>$B2O_w(\overrightarrow x) \doteq -x_{w-1}(2^{w-1}-1) + \sum^{w-2}_{i=0}x_i2^i$</li>
    <h3> ii. Sign Magnitude</h3>
    <li>$B2S_w(\overrightarrow x) \doteq (-1)^{x_{w-1}} (\sum^{w-2}_{i=0}x_i2^i)$</li>
    <li>현대의 거의 모든 Machine은 Two’s-complement를 사용하니 몰라도 될듯?</li>
    <li>Sign-magnitude encoding은 Floating-point Numbers를 표현할 때 사용됨</li>
  </div>
  </details>
  <br>

  

# 2.2.4 Conversions between Signed and Unsigned

## a. C 언어에서의 Data 형 변환

- 변수 `x`를 int로 선언, 변수 `u`를 unsigned로 선언
- `(unsigned) x` : x의 값을 unsigned value로 변환함
- `(int) u` : u의 값을 signed integer로 변환함

### i. 형 변환 시, bit-level의 관점에서 구현됨

```c
short int v = -12345;
unsigned short uv = (unsigned short) v;
printf("v = %d, uv = %u\n", v, uv);
```

- 결과
    
    ![Code Result 2](https://github.com/user-attachments/assets/0ee10b75-fb87-40c2-bbaf-ef7320359d07)
    
    - 두 숫자의 Bit Representation은 [1 1 0 0 1 1 1 1 1 1 0 0 0 1 1 1] 으로 동일함
    - Bit Value는 Identical하고, 이를 어떻게 해석할 것인지만 형 변환을 통해 달라짐.

```c
unsigned u = 4294967295u; /* UMax */
int tu = (int) u;
printf("u = %u, ut = %d\n", u, tu);
```

- 결과
    
    ![Code Result 3](https://github.com/user-attachments/assets/fe13330c-8afc-41b7-baca-44a0fd128aa2)
    
    - $UMax_{32} = 4,294,967,295$ = [ 1 1 1 1 … 1 1 1 1]
<br>

## b. T2U, U2T functions

### i. $T2U_w$ (Two's-complement-to-Unsigned Number)

- $T2U_w(x) \doteq B2U_w(T2B_w(x))$

### ii. $U2T_w$ (Unsigned-to-Two's)

- $U2T_w(x) \doteq B2T_w(U2B_w(x))$

- $T2U_{16}(-12,345) = 53,191$
- $U2T_{16}(53,191) = -12,345$
    - Bit Representation = 0xCFC7
- 12,345 + 53,191 = 65,536 = $2^{16}$ = 1 [ 0 0 0 0 … 0 0 0 0 ]

→ 일반화 : $1 + UMax_w = 2^w$ = [ 0 0 0 0 … 0 0 0 1] + [ 1 1 1 1 … 1 1 1 1] = 1 [ 0 0 0 0 … 0 0 0 0]
<br>

### iii. PRINCIPLE : Conversion from two’s complement to unsigned (T2U)

![PRINCIPLE 2.5](https://github.com/user-attachments/assets/2e64f048-4958-4e3b-8c16-e0dc006ee524)

- DERIVATION
    - $B2U_w(\overrightarrow x) - B2T_w(\overrightarrow x) = x_{w-1}(2^{w-1} - - 2^{w-1}) = x_{w-1}2^w$
    - $B2U_w(\overrightarrow x) = B2T_w(\overrightarrow x) + x_{w-1}2^w$
    - $B2U_w(T2B_w(x)) = T2U_w(x) = x + x_{w-1}2^w$
<br>

### iv. PRINCIPLE : Conversion from Unsigned to two’s-complement (U2T)

![PRINCIPLE 2.7](https://github.com/user-attachments/assets/ec90916f-ad01-4541-8f7a-f0aad7bf448a)

- DERIVATION
    - T2B와 유사
    - $U2T_w(u) = -u_{w-1}2^w + u$



# 2.2.5. Signed versus Unsigned in C

- C standard에서는 Signed Number를 표현하는 방식을 특정하진 않았지만, 대부분의 Machine에서는 Two’s-complement 방식의 표현을 사용함.<br>
→ 또한 Conversion에서도 마찬가지로, C Standard는 특정하지 않았지만 Machine에서는 Bit Representation이 변하지 않는 Rule을 통해 Conversion을 수행함.
- 보통 대부분의 숫자는 Signed가 Default 값임.
    - e.g). 12345 or 0x1A2B를 선언할 때, 기본적으로 Signed로 인식됨.
- ‘U’ or ‘u’라는 suffix를 숫자 뒤에 추가하여 Unsigned Constant임을 명시할 수 있음
    - e.g). 12345U or 0x1A2Bu
<br>

## a. Conversion by Explicit casting and Implicitly Casting

```c
// Explicit Casting
int tx, ty;
unsigned ux, uy;

tx = (int) ux;
uy = (unsigned) ty;
```

```c
// Implicitly Casting
int tx, ty;
unsigned ux, uy;

tx = ux; /* Cast to signed */
uy = tu; /* Cast to unsigned */
```

  
## b. Conversion by Directives from printf

```c
int x = -1;
unsigned u = 2147483648; /* 2 to the 31st */

printf("x = %u = %d\n", x, x);
printf("u = %u = %d\n", u, u);
```

- 결과 as a 32-bit program
    
    ![Code Result 4](https://github.com/user-attachments/assets/5fc0f0f0-92a1-43b3-82fd-0c88c6230a26)
    
    - -1 = [ 1 1 1 1 … 1 1 1 1 ] = 0xFFFFFFFF = UMax
    - 2,147,483,648 = [ 1 0 0 0  … 0 0 0 0 ] = 0x80000000 =  -2,147,483,648

  
### Cf. Format Specifier of printf
<details>
  <summary>Contents</summary>
  <div markdown="1">
  <img width="585" alt="image" src="https://github.com/user-attachments/assets/0288a2e3-d0c6-4f77-86da-02817308b70d">
  </div>
  </details>
    <br>

## c. Effects of C promotion rules

![Figure 2.19](https://github.com/user-attachments/assets/a7810199-43f1-466f-966d-6689d352990b)

- Signed Quantities와 Unsigned Quantities를 동시에 포함하는 Relational Operation을 수행할 때, C는 Signed Argument를 Unsigned Argument로 인식하여 연산을 수행함
- e.g). -1 < 0U
    - -1 = [ 1 1 1 1 … 1 1 1 1 ] = 0xFF..FF
    - 0U = [ 0 0 0 0 … 0 0 0 0 ] = 0x00..00
    - 0xFF..FF를 Unsigned로 인식하여 -1이 UMax가 됨
    → 따라서 False(= 0)
- e.g). 2147583647 > (int) 2147483648U
    - 2147583647 = [ 0 1 1 1 … 1 1 1 1 ] = 0x7F..FF
    - 2147583648U = [ 1 0 0 0 … 0 0 0 0 ] = 0x80..00
        - (int) 0x80..00 = -2147583648
        
        → 따라서 True(= 1)
        

## Cf. Web Aside Data : TMIN (Writing TMin in C)
<details>
  <summary>Contents</summary>
  <div markdown="1">
    <h3> $TMin_{32}$를 표현할 때, 왜 -2,147,483,647 - 1 이라고 표현할까? -2,147,483,648 or 0x80000000라고 안 하고..</h3>
    
    <li> `limits.h` C header file에 $TMin_{32}$와 $TMax_{32}$가 다음과 같이 정의돼 있음.</li>
      <pre><code>
      /* Minimum and Maximum values a 'signed int' can hold. */
      #define INT_MAX 2147483647
      #define INT_MIN (-INT_MAX - 1)
      </code></pre>

    <li> 2의 보수의 비대칭성과 C의 Conversion Rules 때문에, $TMin_{32}$를 보편적인 방식으로 쓰지 않음.</li>
  </div>
  </details>



# 2.2.6. Expanding the Bit Representation of a Number

## a. Expanding the Bit Representation of a Numebr

- 같은 Numeric Value를 갖지만, Word Size는 다른, 두 Integers 사이에서의 Conversion.
- Larger Data type → Smaller Data type은 불가능할 수 있지만,
Smaller Data type → Larger Data type은 항상 가능함.

### i. PRINCIPLE : Expansion of an Unsigned Number by Zero Extension

- Define bit vectors $\overrightarrow u = [u_{w-1}, u_{w-2}, \dots,u_0]$ of width $w$,
and $\overrightarrow u' = [0, \dots , 0, u_{w-1}, u_{w-2}, \dots, u_0]$ of width $w'$, where $w' > w$
- Then $B2U_w(\overrightarrow u) = B2U_{w'}(\overrightarrow u')$

### ii. PRINCIPLE : Expansion of a Two’s-complement Number by Sign Extension

- Define bit vectors $\overrightarrow x = [x_{w-1}, x_{w-2}, \dots,x_0]$ of width $w$,
and $\overrightarrow x' = [x_{w-1}, \dots , x_{w-1}, x_{w-1}, x_{w-2}, \dots, x_0]$ of width $w'$, where $w' > w$
- Then $B2T_w(\overrightarrow x) = B2T_{w'}(\overrightarrow x')$

```c
short sx = -12345;          /* -12345 for 16-bit word size */
unsigned short usx = sx;    /* 53191 for 16-bit word size */
int x = sx;                 /* -12345 for 32-bit word size */
unsigned ux = usx;          /* 53191 for 32-bit word size */

printf("sx = %d:\t", sx);
show_bytes(&sx, sizeof(short));
printf("usx = %u:\t", usx);
show_bytes(&usx, sizeof(unsigned short));
printf("x = %d:\t", x);
show_bytes(&x, sizeof(int));
printf("ux = %u:\t", ux);
show_bytes(&ux, sizeof(unsigned));
```

- 결과 as a 32-bit program on a big-endian machine
    
    ![Code Result 5](https://github.com/user-attachments/assets/dec02800-9a19-4dcc-bceb-f21f92cd84a7)
    
    - Two’s-complement -12,345와 Unsigned 53,191은 16-bit word size에서는 동일하나,
    32-bit word size에서는 다름.
    - 32-bit word size에서,
        - -12,345의 16진수 표현법은 0xFFFFCFC7이지만 (by **Sign Extension**)
        - 53,191의 16진수 표현법은 0x0000CFC7임 (by **Zero Extension**)
    
    - Cf. Big-endian machine?
        - A **big**-**endian** system stores the most significant byte of a word at the smallest memory address and the least significant byte at the largest.

### iii. Figure 2.20 Examples of Sign Extension from w = 3 to w = 4

![Figure 2.20](https://github.com/user-attachments/assets/11fab4a3-8aff-4d6a-a97f-9aba120cd254)

- w = 4로 Sign Extension이 발생할 때, bit vector [1101]의 MSB와 기존(w=3일 때)의 MSB의 연산 결과는 -8 + 4 = -4
- Extension을 수행하기 전, MSB의 값은 -4로
    - 두 경우의 MSB의 Value 연산은 유지됨
- 따라서, Sign Extension은 Value of a Two’s-complement Number를 보존함
- DERIVATION
    
    ![DERIVATION 1](https://github.com/user-attachments/assets/638c3353-2b86-473b-bfd8-a563e759090f)
<br>    

## b. The Relative Order of Conversion from one data size to another, and between Unsigned and Signed

```c
short sx = -12345;  /* -12345 */
unsigned uy = sx;   /* Mystery! */

printf("uy = %u:\t", uy);
show_bytes(&uy, sizeof(unsigned));
```

- 결과
    
    ![Code Result 6](https://github.com/user-attachments/assets/6d095f03-321b-403f-bb5f-a3a6c37085f5)
    
    - short : 2-bytes
    unsigned : 4-bytes
    1. Change the Size (short(2) → int(4)) : -12,345 ( 0xCFC7 ) → -12,345 ( 0xFFFFCFC7 )
    2. Change the Type (int(4) → Unsigned(4)) : -12,345 ( 0xFFFFCFC7 ) → 4,294,954,951 ( 0xFFFFCFC7 )
    - 즉, (unsigned) sx == (unsigned) (int) sx



# 2.2.7. Truncating Numbers (= 숫자 자르기)

## a. Reducing the number of Bits Representation

```c
int x = 53191;
short sx = (short) x;   /* -12345 */
int y = sx;             /* -12345 */
```

- int → short로 Casting하는 것은, 32-bit → 16-bit로 Truncate함
- 결과
    - sx = -12,345 = 0xCFC7 = [ 1 1 0 0 1 1 1 1 1 1 0 0 0 1 1 1 ]
    - y = -12,345 = 0xFFFFCFC7 = [ 1 1 1 1 … 1 1 0 0 1 1 1 1 1 1 0 0 0 1 1 1 ]
    - 이 경우엔 문제 없음

### i. Overflow

- Bit 축소는 원래 숫자의 값을 변경시킬 수도 있음
<br>

### ii. PRINCIPLE : Truncation of an Unsigned Number

- Let $\overrightarrow x = [x_{w-1}, x_{w-2}, \dots, x_0]$, and
let $\overrightarrow x' = [x_{k-1}, x_{k-2}, \dots, x_0]$ (k-bits의 크기로 Truncate함)
- Let $x = B2U_w(\overrightarrow x)$, and $x'= B2U_k(\overrightarrow x')$
- Then $x' = x \space mod \space 2^k$

- e.g). [ 1 1 0 1 ] → [ 0 1 ] (4-bits → 2-bits)
    - [ 1 1 0 1 ] = 8 + 4 + 1 = 13
    - [ 0 1 ] = 1
    - 13 mod $2^2$ = 13 mod 4 = 1
- DERIVATION
    
    ![DERIVATION 2](https://github.com/user-attachments/assets/6439957c-33c1-4e82-a8bc-cae9e2774be2)
    <br>

### iii. PRINCIPLE : Truncation of a Two’s-complement Number

- Let $\overrightarrow x = [x_{w-1}, x_{w-2}, \dots, x_0]$, and
let $\overrightarrow x' = [x_{k-1}, x_{k-2}, \dots, x_0]$ (k-bits의 크기로 Truncate함)
- Let $x = B2T_w(\overrightarrow x)$, and $x'= B2T_k(\overrightarrow x')$
- Then $x' = U2T_k(x \space mod \space 2^k)$

- e.g). [ 1 1 0 1 ] → [ 0 1 ]
    - [ 1 1 0 1 ] = -8 + 4 + 1 = -3
    - [ 0 1 ] = 1
    - $U2T_2(-3 \space mod \space 2^2) = U2T_2(1)$ = [ 0 1 ] = 1
<br>


# 2.2.8. Advice on Signed versus Unsigned

## a. The Implicit Casting of Signed to Unsigned

- 비직관적인 결과로 이어질 수 있고, 또 이것은 Program Bugs를 유발할 수 있음.
- Implicit Casting은 Code에서 명확한 Indication으로 되어 있지 않기 때문에, 찾기도 힘들고 후에 발생할 영향을 간과하는 경우가 많음.

### i. Practice Problem 2.25

```c
/* WARNING: This is buggy code */
float sum_elements(float a[], unsigned length) {
    int i;
    float result = 0;

    for (i = 0; i <= length - 1; i++) {
        result += a[i];
    }
    return result;
}
```

### ii. Practice Problem 2.26

```c
/* Prototype for library function strlen */
size_t sstrlen(const char* s);

/* Determine whether string s is longer than string t */
/* WARNING: This function is buggy */
int strlonger(char* s, char* t) {
    return strlen(s) - strlen(t) > 0;
}
```

1. For what cases will this function produce an incorrect result?
2. Explain how this incorrect result comes about.
3. Chow how to fix the code so that it will work reliably.

## b. Conclusion

- Implicit Conversion으로 인해 발생하는 Bugs를 피하기 위한 방법으로, Unsigned Number를 절대 사용하지 않는 것이 있음.
    - Unsigned Integers의 가치보다 발생시킬 문제들이 더 크다고 생각해서, C가 아닌 다른 보통의 Languages는 Unsigned Integers를 지원하지 않음.
- 하지만 Unsigned Values를 Numeric Interpretation으로 생각하지 않고, 그저 Bits의 Collection으로만 생각한다면 유용할 수 있음.
    - e.g). Describing Boolean Conditions
    - e.g). Addresses are naturally unsigned, etc.

---