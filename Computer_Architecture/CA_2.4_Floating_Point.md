# 2.4 Floating Point

- 부동 소수점(Floating Point) 표현법은 유리수를 인코딩하는 방법이다.
- 거의 모든 컴퓨터가 $V= x*2^y$ 형태의 소수를 IEEE 부동소수점 표현 방식을 통해 지원된다.
## 2.4.1 Fractional Binary Numbers

- 부동 소수점을 이해하기 위해서는 fractional value에 대해서 생각해보아야 한다.
    
    **decimal**
    - $12.34 = 1*10^{1}+ 2*10^{0}+3*10^{-1}+4*10^{-2}$

    **binary**
    - $11.011 = 2^1+2^0+2^{-1}*0+2^{-2}+2^{-3}$
    
![2_4_1](https://github.com/user-attachments/assets/e75dc719-15d4-44c0-9687-b83ce02891e1)
![2_4_2](https://github.com/user-attachments/assets/58b535f0-9b05-42af-aea0-1bfbac684761)

- 위와 같은 방식의 표기법을 보자.
    - 각각의 부호 ‘.’는 decimal point와 binary point이며, binary point의 왼쪽 비트는 2의 거듭 제곱의 값을 가지고, 오른쪽은 2의 음수 거듭 제곱의 값을 가진다.(decimal은 10의 제곱)
    - binary point를 한자리 우측으로 이동하면 2를 곱한 효과이고, 좌측으로 이동하면 2로 나눈 효과를 가진다.(decimal은 10을 곱한 효과)
    - $0.111111_2( \frac{63}{64})$와 같은 수를   1.0 − $\epsilon $ 이라고 표현한다.( $\epsilon $은 매우 작은수  )
- 이진수 표기법은 $x*2^y$로 나타낼 수 있는 수만 표시할 수 있다.
- 이진수 표기법으로 나타내기 매우 힘든 수도 존재한다. 
    - ex) 1/5의 fractional decimal number는 0.20 이며 이를 binary representation으로 나타내기 힘들다.
- 따라서 이진수 표기의 길이를 늘려 정확성을 높이도록 근사해야 한다.

## 2.4.2 IEEE Floating - Point Representation

- IEEE 부동소수점 표준은 $V=(-1)^s * M * 2^E$ 의 형태로 나타낸다.
    - s는 부호를 나타낸다.
    - M은 유효숫자로 분수 이진 값이다.
        - m은 0~ 1.0 − $\epsilon $ 또는 1 ~ 2.0 − $\epsilon $ 값을 가진다.
    - E는 지수로 $2^E$은 2의 제곱 값을 갖는다.
- 부동소수점의 비트 표현은 세 개의 필드로 나누어진다.
    - 한 개의 부호 비트 s는 부호 s를 인코딩한다.
    - k비트 exp는 지수 E를 인코딩한다.
    - n비트 frac은 유효숫자 M을 인코딩한다.

![2_4_3](https://github.com/user-attachments/assets/83b02aed-b708-4101-ac8f-f76a11e41440)
- single-precision(c. float) 은 s = 1, k = 8, n = 23 bit이며 총 32bit이다.
- double-precision(c. double) 은 s=1 , k=11, n=52 bit이며 총 64bit이다.


---     
![2_4_4](https://github.com/user-attachments/assets/c2bf705a-9c17-4e2d-a449-c19a2124a78d)
- 비트 표현법은 exp의 값에 따라 세 가지 경우가 있다.


    **Case 1 : Normalized Values**
    - 가장 일반적인 경우
    - exp의 비트  : 모두 0이 아니거나 모두 1이 아니어야 한다.
    - E = e - Bias(Bias = $2^{k-1}-1$)
        - sign number에서 큰 수가 더 크게 보여 비교를 더 용이하게 하기 위해 사용
        - single-precision 에서 -126에서 +127의 범위를 가진다.
    - frac 필드는 0≤f<1까지의 범위를 갖는 분수 값을 표현한다.
    - 유효 숫자 M = 1+f
        - 더해진 1을 *implied leading 1* 이라고 불린다.
        - f에 1을 더하는 이유는 binary 표현법에서 첫 bit는 항상 1이기 때문에 생략하여 bit 한 개를 아끼는 용도이다.
        -  $±1.xxxx_2× 2^{yyyy}$

    **Case 2 : Denormalized Values**
    - 지수 필드가 모두 0일 때를 말한다.
    - E = 1 - Bias
    - M = f
    - Denormalized value의 목적은 두 가지 이다.
        1. 숫자 0을 표현
            - normalized value는 M≥1이므로 0을 표현할 수 없다.
            - 모든 비트가 0이면 +0.0, sign bit만 1이면 -0.0을 표현한다.
        2. 0의 근사치 표현
            - 0.0에 매우 가까운 값들을 나타낸다.

    **Case 3 : Special Values**
    - ∞, NaN을 표현하기 위한 값
    - exp 필드가 모두 1
        - frac 필드가 모두 0이면 ∞ 이며, s= 0 이면 +∞, s=1이면 -∞
        - frac 필드가 0이 아니면 NaN(not a number)
            - NaN
                - $\sqrt{-1}$ , ∞ - ∞ 와 같이 실수나 ∞으로 나타내지 못할 떄

## 2.4.3 Example Numbers

![2_4_5](https://github.com/user-attachments/assets/8583bebe-2e2e-4c7f-a73b-6d118cafc346)
- normalized number는 골고루 분포되어 있지만 denormalized number는 0에 가까운 곳에 밀집되어 있다.
- 총 6bit, k = 3bit, n=2bit일 때 최대값을 구해보시오 -> +14
- 총 8bit, k = 4bit, n = 3bit일 때 최대값을 구해보시오. -> +240

![2_4_6](https://github.com/user-attachments/assets/9bb7b430-c0f8-41c1-b7d5-b3fa4b3be26c)
- largest denormalized와 smallest normalized의 값이 부드럽게 이어지는 이유는 E의 정의 덕분이다. E를 1−Bias로 정의함으로써, 비정규화된 숫자의 유효숫자가 *implied leading 1*을 가지지 않는다는 사실을 보정합니다.
    - denormalized 가 최대 값 : 
    
        $f = 0.11111.._2 = 1-2^{-23}=0.9999988..$

        $V = M*2^E = 0.9999988_2^{-126}$
    - normalized 가 최소 값:

        $M = 1.00000_2$ (*implied leading 1* 포함  )

        $V = M*2^E = 1.0000_2^{-126}$
## 2.4.4 Rounding

- 부동소수점 산술 연산은 범위와 정밀도가 제한되기 때문에 실제 값에 근사할 뿐이다.
- 가장 근사한 값을 찾는 체계적인 방법이 rounding 이다.
- 주요 문제점은 두 가지 가능성을 갖고 있는 값에 대한 반올림의 방향성이다.
    - ex) 1.5 → 1 or 2
- IEEE 부동 소수점은 4가지 모드를 정의한다.
    - round to even : 가장 가까운 값, 마지막 자리 수가 짝수를 향하게 근사함
    - round toward zero : 0 방향으로 근사, 양수이면 아래, 음수이면 위 방향
        - $∣\hat{x}∣≤∣x∣$
    - round down : 모두 아래로 근사
        - $x^{-1}≤x$
    - round up : 모두 위로 근사
    - $x≤x^+$
    
![2_4_7](https://github.com/user-attachments/assets/6934d74c-9ff4-404e-8eb9-68be2a8a896e)
- round to even 이 가장 기본적인 방법이다.
    - 이유: 통계적 편향을 피할 수 있다.
        - round down이나 round up을 사용하여 근사했을 때 값들의 평균값은 원래 값보다 작거나 같게 나올 것이다.
        - round to even을 사용하면 약 50%는 아래로, 나머지 50%는 위로 근사하기 때문에 통계적 편향을 피할 수 있다.
- 이진 소수에서의 round to even 사용
    - XX...X.YY...Y100...
    - Y에서 반올림 할 때 Y 뒤의 값이 100…일 때 정확히 중간의 값을 가진다.
    
        ex)
        - $00.11100_2(7/8)$ → $01.00_2(1)$
        - $00.10100_2(5/8)$ → $00.10_2(1/2)$
        - 마지막 bit가 0이 되도록 round

## 2.4.5 Floating - Point Operations

- Floating point operation은 교환법칙은 성립하지만, unsigned , two’s complement number operation과 다르게 결합 법칙이 성립하지 않는다.

    ex) 
    - (3.14+1e10)−1e10을 계산하면 3.14가 반올림되기 때문에 0이 나온다.
    - 3.14+(1e10−1e10)을 계산하면 3.14가 반환된다.
- unsigned, two’s complement는 만족하지 않는 단조성(monotonicity)를 만족한다.
    - 단조성이란
        - a≥b일 때, NaN이 아닌 모든 a,b,x에 대해
            
            $x+_fa≥x+_fb$ 
            
            을 만족하는 것
            
        - unsigned와 two’s complement는 만족하지 않는다.

## 2.4.6 Floating Point in C

- C는 두 가지 다른 부동 소수점 data type인 float와 double을 제공한다.
- round to even을 사용한다.
- C 표준에서는 기계가 IEEE 부동 소수점을 사용할 것을 요구하지 않기 때문에 round mode를 변경하거나 -0, ∞, NaN과 같은 특수 값을 얻기 위한 표준 방법은 없고, .h파일과 라이브러리 함수를 제공한다.
- data type casting
    - int → float
        - 오버플로우되지 않지만 반올림 될 수 있다. (int가 정밀도는 더 높다.)
    - int, float → double
        - double이 더 큰 범위 이기 때문에 정확한 값을 보존한다.
    - double → float
        - 범위가 더 작기 때문에 오버플로우 될 수 있고, 반올림 될 수 있다.
    - float, double → int
        - 오버플로우 될 수 있고, 0쪽으로 근사된다.