---
up: "[[CA_2_Representing_and_Manipulating_Information]]"
related: 
tags: 📝/🌱️
aliases: 
cssclasses:
  - dashboard
created: 2024-09-28T18:37
modified: 2024-10-03T19:13
---

<img width="585" alt="image" src="https://github.com/user-attachments/assets/f643bd41-9da6-4c87-a9ac-575d08b23048">

-   컴퓨터는 메모리를 개별 비트로 접근하지 않고 바이트(8비트)의 블록을 메모리의 가장 작은 주소 단위로 사용한다.
-   실제로 메모리는 비트 단위로 존재하지만, 기계 입장에서는 메모리를 바이트의 매우 큰 배열로 인식하는데 이를 가상 메모리(virtual memory)라고 한다.
-   메모리의 각 바이트는 고유한 번호로 식별되고, 이를 주소(address)라 한다.
-   모든 주소의 집합은 가상 주소 공간(virtual adress space)라 한다.

<br>

---

## 2.1.1 Hexadecimal Notation

![image](https://github.com/user-attachments/assets/d09a9b47-ed44-4e20-8618-06a0fc8eaa9e)

-   1 바이트 = 8 비트
-   2진법(Binary Number) : $00000000_2$ ~ $11111111_2$
-   10진법(Decimal) : 0 ~ 256($2^8$)
-   16진법(Hexadecimal) : $00_{16}$ ~ $FF_{16}$
-   16진법은 0 $\sim$ 9, A, B, C, D, E, F 를 사용하여 10진법의 0 $\sim$ 15 숫자를 표현한다.

### Translation

#### Translate between binary to hexadecimal

-   2진수에서 16진수로 변환할 경우 4개의 비트로 쪼개어 16진수로 변환한다.
-   16진수에서 2진수로 변환할 경우 한 문자당 4비트씩으로 표현하면 된다.
-   1111001010110110110011 변환해보기

-   $x=2^n$ 일 경우 x를 16진수로 변환할 때는 다음 규칙을 사용한다.
    -   $n = i + 4j$ $(0 <= i <= 3)$
    -   첫 자리수는 $2^i$ 이다.
    -   첫 자리수 뒤에 j 개의 0을 붙인다.
    -   ex) $x = 2^7$
        -   $7 = 3 + 4 \times 1$, ($i = 3$, $j=1$)
        -   result = 0x80

![image](https://github.com/user-attachments/assets/bf7f9161-f9a8-4eae-a3b8-a71eee3f4b76)

-   n의 변화에 따라 규칙이 존재한다.

#### Translate between decimal and hexadecimal

-   10진수 숫자 x를 16진수로 변환
    -   x를 16으로 나눠 몫 q, 나머지 r을 얻는다.
    -   $x=q⋅16+r$
    -   q가 0이 될 때 까지 나누고, 나머지의 숫자를 역순으로 나열한다.
    -   아래의 경우 0x4CB2C 가 된다.

$$
\begin{aligned}
314,156 = 19,634 . 16 + 12 (C) \newline
19,634 = 1,227 . 16 + 2 (2) \newline
1,227 = 76 . 16 + 11 (B) \newline
76 = 4 . 16 + 12 (C)\newline
4 = 0 . 16 + 4 (4)
\end{aligned}
$$

-   16진수 숫자 y를 10진수로 변환
    -   각 16진수 숫자를 16의 거듭제곱으로 곱한다.
    -   $y = 0x1AF$
    -   $1 \cdot 16^2 + 10 \cdot 16 + 15 = 1 \cdot 256 + 10 \cdot 16 + 15 = 256 + 160 + 15 = 431$

<br>

---

## 2.1.2 Data Sizes

-   모든 컴퓨터는 워드 크기(Word Size)를 갖는다.

    -   워드 크기 : 컴퓨터 시스템에서 한 번에 처리할 수 있는 데이터 크기
    -   워드 크기는 가상 주소 공간의 최대 크기를 결정한다.
        -   w-비트 워드 크기 ==> 가상 주소 공간 범위 $0 \sim 2^w-1$
        -   최대 $2^w$ 바이트에 접근 가능

-   과거에는 32비트 머신을 사용했는데, 이는 곧 가상 주소 공간의 최대 크기가 $2^{32}$ 바이트(4기가바이트, $4\times 10^9$)임을 나타낸다. (가상 주소 공간도 4GB 밖에 안 된다.)
-   가상 주소 공간 부족으로 인해 64비트 머신으로 전환이 이루어지면서 가상 주소 공간의 최대 크기가 $2^{64}$ 바이트(16 엑사바이트, $1.84\times 10^{19}$ 바이트)로 증가했다.
    -   32비트 컴파일 프로그램은 64비트에서 돌아가지만, 64비트 컴파일 프로그램은 돌아가지 않는다.(32 -> 64 마이그레이션 하는 과정에서 워드 크기 의존성 버그가 많았음)

<img width="612" alt="image" src="https://github.com/user-attachments/assets/94f04e96-519a-4314-8d17-886137ffe65c">

-   머신에 따라 지정한 데이터 타입의 바이트가 다르다.
    -   int32_t, int64_t 의 경우 명확하게 각각 4, 8 바이트로 설정하여 32비트 머신, 64비트 머신에서도 통용될 수 있도록 한다.(해당 타입을 사용해 더 정확한 표현을 할 수 있다.)
-   signed char : 보통 char의 경우 부호를 신경쓰지 않는다.
    -   다만, signed를 사용함으로써 부호 있는 1바이트 정수를 사용해야 함을 보장할 수 있다.
    -   C 언어에서 char 데이터 타입
        -   일반적으로 생각하는 character, 문자 표현도 있으나, 객체 표현(원시 메모리)를 검사하는데 사용된다.
-   char \* : 포인터는 프로그램 전체의 워드 크기를 사용한다.

<br>

---

## 2.1.3 Addressing and Byte Ordering

-   전체적인 데이터 크기는 워드 크기로 설정된다.
-   다음 생각할 것은 "데이터를 메모리에 실질적으로 저장할 때는 어떻게 저장할래?" 다.
-   대부분 바이트 뭉치는 연속된 순서의 주소에 저장된다.

<img width="492" alt="image" src="https://github.com/user-attachments/assets/60a9feaa-43cd-474e-99c2-f5ecb5c40aa7">

-   int32_t로 선언된 변수 x의 경우 4바이트이고, 0x01234567 값이라 할 경우
    -   1바이트씩, 16진수 2자리씩 저장된다.
-   그러면 순서는 어떻게 저장할 건데?

    -   Big endian : 자리수가 큰 것 부터 저장하자
        -   장점 1. 우리가 읽는 방식으로 저장됨 -> 리버싱, 디버깅 수월함
        -   장점 2. 수를 비교할 때 큰 수부터 비교하고, 큰 수가 앞에 있기에 편함
        -   사용처 : IBM, 오라클, 네트워크 통신 시
    -   Little endian : 자리수 작은 것 부터 저장하자
        -   장점 : 덧셈 연산을 하는 경우 낮은 자리부터 연산하기 때문에 더 빠르다.
        -   사용처 : Intel, Android, iOS

-   바이트 순서가 문제가 되는 경우
    1.  네트워크 통신 간 서로 다른 방식을 사용하는 경우 -> 바이트 순서가 반대로 나타남
    2.  정수 데이터를 나타내는 바이트를 읽어야 할 때(리버싱, 디버깅 등)
    3.  일반적인 타입 시스템을 우회하는 프로그램을 작성할 때
        -   캐스트(cast)나 유니온(union)을 사용해 선언한 타입과 다른 타입에 따라 참조할 수 있다.
        -   데이터 형태를 변경해 작업을 해야하는 경우 (메모리의 바이트 표현, 메모리 조작 등)
        -   다양한 데이터 형식을 운용해야 할 경우

```
#include <stdio.h>

typedef unsigned char *byte_pointer;

void show_bytes(byte_pointer start, size_t len) {
	int i;
	for (i = 0; i < len; i++)
		printf(" %.2x", start[i]);
	printf("\n");
}

void show_int(int x) {
	show_bytes((byte_pointer) &x, sizeof(int));
}

void show_float(float x) {
	show_bytes((byte_pointer) &x, sizeof(float));
}

void show_pointer(void *x) {
	show_bytes((byte_pointer) &x, sizeof(void *));
}
```

-   unsigned char \* 로 캐스팅하여 메모리 주소를 바이트 표현으로 접근할 수 있도록 한다.
-   show_bytes 함수를 통해서 데이터 타입 길이만큼 메모리 주소값을 출력한다.
-   이와 같이 출력할 때도 순서를 고려해야 한다.

<img width="622" alt="image" src="https://github.com/user-attachments/assets/29c6b602-30ec-493e-a19f-1fcca68e852c">

-   `int x = 12345` 를 float, int \* 로 형 변환한 후 메모리를 출력한 표이다.
-   같은 12345를 출력하지만, 출력되는 메모리 표현은 차이가 나는데, 이는 부동소수점 표현과 정수 데이터 표현의 차이로 인해 발생한다.
-   각 운영체제마다 저장하는 순서의 차이가 발생한다.
-   포인터의 경우 각 운영체제마다 정의하는 방식이 모두 다르다.

<br>

---

## 2.1.4 Representing Strings

-   문자의 표현은 특정 표준 인코딩으로 표현하는데, 주로 ASCII 코드를 통해서 인코딩한다.
-   C에서 문자열은 null(값이 0)을 표현해 문자를 종료한다.
-   "12345" 문자열(len=6)을 통해 31 32 33 34 35 00 의 결과를 얻는다.
    -   null값으로 인해 길이가 1증가한 6이 된다.
    -   10진수 숫자 x의 ASCII 코드값은 3x 다.
-   워드 크기, 바이트 순서에 관계 없이 ASCII 코드를 사용하는 시스템에서는 모두 동일하다.

> [!Question] 왜 다른 규약들은 통일하지 않았을까?
>
> 1. 각기 다른 시스템과 환경에서 개발된 프로그램들이 서로 다른 요구 사항과 설계 목표를 가졌음.
> 2. 특정한 기술적 필요나 역사적 배경에 따라 서로 다른 표준과 규약이 발전했음.

<br>

---

## 2.1.5 Representing Code

```c
int sum(int x, int y){
	return x + y;
}
```

-   위 코드를 각 운영체제마다 컴파일하는 경우 바이트 표현은 어떻게 될까?
    -   모두 다르다.
    -   운영체제는 서로 호환되지 않는 명령어와 인코딩을 사용한다.
    -   같은 프로세서를 사용하더라도 다른 운영체제를 사용하면 이진 코드는 서로 다르게 표현된다.

<br>

---

## 2.1.6 Introduction to Boolean Algebra

-   불 대수(Boolean algebra) by George Boole
    -   논리 값 True, False를 1, 0으로 인코딩하여 정보를 조작한다.

<img width="611" alt="image" src="https://github.com/user-attachments/assets/1f272a57-cb28-40f4-8b7f-6b4af76715d8">

-   불 대수를 비트벡터에 적용할 수 있다.

<img width="404" alt="image" src="https://github.com/user-attachments/assets/30e2057f-d98c-407b-adab-104620ec55f6">

-   비트 벡터와 불 대수를 통해서 유한 집합을 표현할 수 있다.
    -   a = [01101001], A = {0, 3, 5, 6}
    -   b = [01010101], B = {0, 2, 4, 6}
    -   a&b = [01000001], A Union B = {0, 6}

<br>

---

## 2.1.7 Bit-Level Operations in C

-   C 언어에서는 비트 단위의 불 논리 연산을 지원한다.
-   ~, &, |, ^ 를 사용한다.

<img width="580" alt="image" src="https://github.com/user-attachments/assets/7a4c8a33-e8fb-401e-981d-9e7fa9d3ac9f">

#### 비트 연산 예시

1. 마스킹: 특정 비트를 선택하거나 무시하기 위한 비트 마스크를 사용하는 경우.
    - 특정 비트를 1로 설정하거나 0으로 설정하기 위해 AND, OR, XOR 연산을 사용.
    - x = 0x12345678
    - x & 0xFF = 0x00000078 (마스킹)
2. 플래그 설정 및 검사: 여러 상태를 하나의 변수에 저장할 때 비트 플래그를 사용. 이를 통해 메모리 사용을 효율적으로 관리할 수 있다.
    - [0001] => [온도센서A, B, C, D의 경우]
3. 데이터 압축: 비트 연산을 통해 특정 데이터를 더 작은 크기로 압축할 수 있다.
    - RGB 색상을 비트 단위로 표현하여 색상 정보를 효율적으로 저장할 수 있다.

<br>

---

## 2.1.8 Logical Operations in C

-   C 언어는 논리연산자 ||, &&, ! 을 제공한다.
-   비트 연산자와의 차이점은 값은 나타내는 것이 아닌, 참/거짓을 나타낸다.
    -   0을 False, 0이 아닌 것을 True로 나타낸다.

<img width="194" alt="image" src="https://github.com/user-attachments/assets/d3b04f0b-b490-4bcb-b12a-7ebc2fd6cbd9">

-   비트 연산, 논리 연산을 사용해서 x == y 를 표현해보기
    -   x 와 y가 같으면 1 다르면 0

<br>

---

## 2.1.9 Shift Operations in C

-   C 언어는 이동 연산자 >>, << 을 제공한다.

#### << 의 경우

-   왼쪽으로 이동시키는 연산자이다.
-   x = [01110010]
-   x << k 표현을 통해 x를 왼쪽으로 k번 이동시킨다.
-   이동시켰을 때 밀리게 되는 높은 값을 가지는 중요한 비트 k 개가 제거된다.
-   오른쪽 끝은 k개의 0으로 채워진다.
-   x << 3 == [10010000]
-   x << j << k 는 (x << j) << k 와 동일하다.

#### >> 의 경우

-   오른쪽으로 이동시키는 연산자이다. 2가지 형태로 작동한다.
-   x >> k 의 경우
    -   논리 이동(Logical) : 왼쪽에는 k개의 0을 채우고 오른쪽으로 k만큼 민다.
    -   산술 이동(Arithmatic) : 왼쪽에는 MSB(Most Significant Bit)를 k개 채우고 오른쪽으로 k만큼 민다.
        -   부호값에 신경써야 한다.(부호값에 따라간다.)
-   언제 논리/산술 이동이 사용되어야 하는지에 대해서는 정확하게 정의되어 있지 않지만 대부분은 다음을 따른다.
    -   부호 있는 데이터의 경우 산술 이동을 사용한다. (대부분 이렇게 가정한다.)
    -   부호 없는 데이터의 경우 반드시 논리 이동을 사용한다.

#### k 이동에서 k 값이 큰 경우

-   w 비트의 데이터 타입을 가질 때, k >= w 이면 이동은 어떻게 될까?
-   int 데이터 타입으로 w=32라 하자.

```c
int lval = 0xFEDCBA98 << 32; // => 0xFEDCBA98
int aval = 0xFEDCBA98 >> 36; // => 0xFFEDCBA9
unsinged int uval = 0xFEDCBA98 >> 40; // => 0x00FEDCBA
```

-   다음과 같이 데이터 타입보다 크게 이동할 경우에 대해 명확하게 명시하지는 않았으나, 보통 k mod w로 계산된다.
-   즉 0, 4, 8 만큼 이동하는 것처럼 계산되어 영향을 준다.
-   부호가 있는 경우 산술 이동, 부호가 없는 경우 논리 이동을 사용한다.

#### 이동 연산자의 연산자 우선순위

-   1 << 2 + 3 << 4 인 경우
    -   (1 << 2) + (3 << 4) 를 의미한다 생각할 수 있지만 그렇지 않다.
    -   (1 << (2 + 3)) << 4 를 의미한다.
    -   따라서 예상한 결과값은 52이지만, 실제 결과값은 512가 된다.
    -   연산자 우선순위를 보다 명확하게 지정하기 위해서는 괄호를 사용해야 한다.
