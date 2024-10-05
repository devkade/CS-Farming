# 2.3 Integer Arithmetic

- 컴퓨터 세상에서는 두 개의 positive num를 더했을 때, negative number result가 될 수 있고, $x<y $와 $x-y<0$ 의 결과가 다를 수 있다.

- $x,y$를 w-bit의 number라고 하며, $x = 2^w-1, y = -1$일 때 

→ $x<y$ 는 false이고, 
→ $x-y<0$에서는 $2^w-1 -(-1) = 2^w $일 때 $2^w$는 overflow되어 음수가 되기 때문에 $x-y<0$ 은 true이다.

## 2.3.1 unsigned addition

- lisp 같은 일부 프로그래밍 언어는 임의 크기의 정수를 허용하는 무한 정수 연산이지만, 대부분의 언어는 고정 크기의 정수 연산을 지원한다.


- 두 개의 음이 아닌 정수 x와 y $0≤x, y<2^w$  는 w bit로 표현된다. 
- 두 수의 합의 범위는 $0≤x+y≤2^(w+1)-2$가 된다. 이는 w+1bit를 필요로 한다.  
            
    → 따라서 w+1 bit를 w bit로 변환해야 할 필요가 있다.
- $ x +^{u}_{w} y $  정의(w bit의 unsigned number 더하기) : x+y의 결과를 w bit로 자르고 그 결과를 부호 없는 숫자로 해석한 결과.

![2_3_2 11](https://github.com/user-attachments/assets/09f7619a-9d65-4e93-8818-dfacbbbe64ae)

![2_3_2 1](https://github.com/user-attachments/assets/fd4fbeef-1f28-4ba7-aafe-03bf83e5bbed)
- **DERIVATION**

    → $ 2^w ≤ x + y < 2^{w+1} $ 일 때는 w+1 bit는 항상 1이다. 따라서 가장 마지막 bit를 지우는 것은 $2^w$를 빼는 것과 같다.
- (e.g. if w = 4bit) 
→ x= 15, y = 8 일 때 x, y의 bit representation은 [1111], [1000] 이고 두 수의 합은 [10111]이다. 이 때 상위 bit를 버리면 [0111]로 7이 된다. 이는 (15 + 8) mod 16과 일치한다. 따라서 $ x +^{u}_{4} y $ = 7이다.


### **Detecting overflow of unsigned number**

- 0≤ x, y≤ $UMaxw$ 일 때, s = $ x +^{u}_{w} y $ 라고 하자.  이 때 s < x 이거나 s < y일 때 overflow가 발생한 것을 알 수 있다.
- **DERIVATION**
    
    만약 overflow가 발생하였다면 $ s= x+y-2^w$가 된다. 이 때 $y-2^w<0$이므로 $s= x+(y-2^w) <x$

### unsigned negation

- modular addition 은 수학적 구조인 아벨 군을 형성하며(교환법칙이 성립되는 군), 모든 요소는 addition 했을 때 0이 되는 덧셈의 역원을 가진다.
    - 군은 어떤 집합에 이항연산이 주어진 구조
    - 군의 특징은 결합 법칙, 항등원의 존재, 역원의 존재를 만족한다.
    - 항등원은 덧셈에서는 0이다. ($a + e = a$ 일 때 e를 항등원)
    - 역원은 $-^{u}_{w}x$ 이다. ($a + x = e$일 때 x를 역원)
- w비트의 unsigned number 집합과 덧셈 연산 $+^{u}_ {w}$ 에 대해 역원이 존재해야 하기 때문에, 각 값 x에 대해 $-^{u}_{w}x$ 라는 어떤 값이 존재해야 하며, $-^{u}_{w}x$ $+^{u}_ {w}$ = 0이어야 한다.

![2_3_2 12](https://github.com/user-attachments/assets/2bd83792-c5c7-4f44-b2a3-510fa11f752f)

- **DERIVATION**

    →  x= 0 일 때, 덧셈 역원은 0,  x > 0 일 때, $2^w - x$ 값은 $0 < 2^w - x < 2^w$ 범위에 존재하며,$( x+ 2^w -x ) mod 2^w = 0$이다. 따라서 $+^{u}_{w}x$의 역원은 $2^w - x$이다.

## 2.3.2 Two’s complement additon

- Two’s complement additon에서 결과가 너무 크거나 너무 작을 때 처리하는 방법
- 정수 x, y 의 범위 $-2^{w-1} ≤ x, y ≤ 2^{w-1}-1$ 일 때, 두 수의 합의 범위는 $-2^w ≤ x+y ≤ 2^w-2$이다. 이를 표현하기 위해서는 w+1 비트가 필요하다.
- $x + ^{t}_{ w} y$ : 정수 합 x+ y를 w 비트로 자른 후 그 결과를 2의 보수 숫자로 보는 것.

![2_3_2 13](https://github.com/user-attachments/assets/2a69fb29-3251-4541-87fe-ea262e8bd16f)

- **DERIVATION**

    - unsigned 와 two’s complement addition은 정확히 같은 bit-level representation을 가지기 때문에, 대부분의 컴퓨터는 두 addition 연산에 대해 같은 기계 명령어를 사용한다

    
        → 같은 bit-level representation 이므로 $x + ^{t}_{ w} y$ 는 x, y를 unsigned number 로 변환 후  $x + ^{u}_{ w} y$ 수행 후 다시 two’s complement number로 변환하는 것과 같다. 
        - 이를 식으로 나타내면
        
        ![2_3_2 14](https://github.com/user-attachments/assets/a4d74b2a-b1b3-49f4-8c1d-cffd1eff34d6)
        

    - $T2U_w(x)$는  식 2.6에 의해 $T2U_w(x) = x_{w-1}*2^w+x$ 이므로 위의 식을 정리하면

        → $x + ^{t}_{ w} y = U2Tw[(x+y)mod 2^w]$
        
   
![2_3_2 2](https://github.com/user-attachments/assets/3da1436e-1d11-4d31-9e03-5f3709a92ae5)
    
    → x+y의 4가지(Case 1~4) 범위에 따라 각각의 값으로 매핑된다.
 - ex)    Case 1 ($-2^{w}≤x+y<-2^{w-1}$) 
    - $(x+y) mod2^w = x+y +2^w$
    - $x+y+2^w$ ≤ $Tmax_w$ 이므로 $U2Tw[x+y+2^w] = x+y+2^w$

![2_3_2 18](https://github.com/user-attachments/assets/de8a1468-1c12-4e6b-b6e1-1242e8f7c444)
### **Detecting overflow int two’s complement addition**

- $Tmin_w≤x,y≤TMax_w$ 일 때, $s = x + ^{t} _{w} y$라 할 때, 양의 오버플로우는 x>0, y>0 이지만, s≤0일 때 발생, 음의 오버플로우는 x<0, y<0이지만 s≥0일 때 발생.
- x, y의 부호가 다를 때는 오버플로우 x

## 2.3.3 Two’s complement Negation

- unsigned 와 마찬가지로 $+ ^{t} _{w} x$ 의 역원이 존재한다.

![2_3_2 15](https://github.com/user-attachments/assets/a0798146-04bf-4eed-bfa8-3de3da42c484)

- $x > TMin_w$인 경우, 값 −x는 w비트 2의 보수 숫자로도 표현될 수 있으며, 그들의 합은 $−x + x = 0$이 된다.
- $x=TMinw$인 경우, $TMin_w + TMin_w = −2^w−1 + −2^w−1 = −2^w$이므로 음의 오버플로우가 발생한다. 따라서 $TMin_w +^{t} _{w}  TMin_w = −2^w + 2^w = 0$이 된다. $(Case1 : x+y+2^w)$

## 2.3.4 Unsigned Multiplication

- 정수 x, y가 $0≤x, y ≤ 2^w-1$ 범위에 있을 때 , 이들의 곱 $x*y$ 는 $0≤ x*y ≤(2^w-1)^2 = 2^{2w}-2^{w+1} +1$ 의 값을 가질 수 있으므로 최대 2w 비트가 필요하다.
- C언어에서는 하위 w bit 결과를 반환하도록 정의되어있으며, 이는 $mod 2^w$와 같다.  이를 $x*^{u}_ {w} y$라고 한다.

![2_3_2 16](https://github.com/user-attachments/assets/d3edc8cf-9ce5-413d-80d7-c610efeca5fe)


## 2.3.5 Two’s complement Multiplication

- 정수 x, y가 $-2^{w-1} ≤ x,y ≤ 2^{w-1}-1$ 범위에 있을 때, 이들의 곱 $x*y$는 $-2^{2w-2}+2^{w-1} ≤ x*y≤ 2^{2w-2}$ 의 값을 가지므로 최대 2w 비트가 필요하다.
- C언어에서는 하위 w bit 결과를 반환하도록 정의되어있으며,  이는 $2^w$로 나눈 나머지를 구한 다음, unsigned number 에서 two’s complement number로 변환하는 것과 같다. ( $x+^{t} _{w}y$ 와 유사하게 bit-level representation이 같기 때문에)


![2_3_2 17](https://github.com/user-attachments/assets/970cf25c-ebf6-47f6-9c36-e32ccfb27eee)

## **Bit level equivalence of unsigned and two’s complement multiplication**

- $x = B2Tw(x), y= B2Tw(y)$라 하고, $x' = B2Uw(x), y' = B2Uw(y)$라 할 때, $T2Bw(x*^{t} _{w}y) = U2Bw(x'*^{u} _{w}y')$ 이다. 
    
    - w bit로 자른 각각의 $x*y$ 값을 binary로 바꾸었을 때 값이 같다.  ---> bit level equicalence하다. 
- **DERIVATION**
    ![2_3_2 3](https://github.com/user-attachments/assets/ba10d381-4655-4b0d-96bd-5c3c306cbbf4)


## 2.3.6 Multiplying by Constants

- 정수 곱셈 명령은 10개 이상의 클럭 사이클을 요구할 만큼 매우 느리지만, 덧셈, 뺄셈, 비트 연산, 시프트 연산 등 다른 정수 연산은 한 클럭 사이클만 필요하다. 
- 이러한 이유로 컴파일러는 상수 곱셈을 시프트와 덧셈 연산의 조합으로 대체하는 최적화를 진행하였다.
- 예를 들어, 11을 4비트로 표현하면 1011이다. 2만큼 왼쪽으로 shift하면 101100으로 44가 된다. 왼쪽으로 값을 shift하는 것은 2의 거듭 제곱으로 부호 없는 곱셈을 하는 것과 같다.

### **unsigned, two’s complement  multiplication by a power of 2^w**

- 2의 보수와 unsigned는 bit level이 동일하므로 상수 곱셈에 대해 유사한 주장을 할 수 있다.
- 만약 $x*14$ 일 때 $14 = 2^3+ 2^2 + 2^1$ 로 인식하여 $(x<<3) + (x<<2) + (x<<1)$ 과 동일하다. 이는 오버플로우가 발생하더라도 2의 보수와 unsigned 상수 곱셈은 동일한 결과를 생성한다.

- **형식 A**: $(x << n) + (x << (n−1)) + ... + (x << m)$
- **형식 B**: $(x << (n+1)) - (x << m)$
- 하드웨어 명령 실행 속도와 연산 식에 따라 최적화의 효율성은 달라질 수 있다.

# 2.3.7 Dividing by powers of 2

### **Unsigned division by a power of 2**

- 2^k으로 나누고 0으로 반올림하는 것은 오른쪽으로 k만큼 logical shift하는 것과 같다.
- 이 때 반올림은 rounds toward zero이다.

![2_3_2 5](https://github.com/user-attachments/assets/568b7ca1-705a-4604-a571-66d65685c36e)

### **Two’s complement division by a power of 2, rounding down**

- 2의 보수 산술을 사용한 경우 , 음수 값이 음수로 유지되도록 arithmetic shift를 수행해야 한다.
    - **SHIFT**
        
        Shift의 종류에는 left shift와 right shift가 있고, 이 때 right shift에는 logical shift와 arithmetic shift가 있다.
        -   logical shift
            - shift하고 남은 공간을 단순히 0을 채워 넣는다.
        - arithmetic shift
            - MSB(Most significant bit)를 복제하여 sign bit에 넣고 빈공간을 sign bit로 채워 넣는다.
    - right에만 arithmetic shift를 사용하는 이유는 arithmetic left shift를 사용하면 overflow가 발생할 수 있다.
        - (e.g. [1001 1100] →[0011 1001]→ overflow)




![2_3_2 4](https://github.com/user-attachments/assets/6be60ef9-01b6-470e-97cb-ed292ee50243)

- arithmetic shift를 사용하면 logical shift와 다르게 round toward zero가 아니라 round downward(내림)이다.

    - ⌊x / 2^k⌋  에서 x/2^k가 음수 일 때 작거나 같은 정수이기 때문에 0보다 먼 수로 반올림 된다.


![2_3_2 6](https://github.com/user-attachments/assets/6292f05c-4987-461c-8472-658d2d01e6fa)

- 책에서는 rounding down을 improper한 rounding이라는 표현을 사용한다.
- proper한 rounding을 하기 위해서는 x/y<0 일 때는 $\lceil x/y \rceil$, x/y>0 에서는 $\lfloor x/y \rfloor$ 가 되어 round toward zero가 되어야 한다. 
- 이는 bias를 더하여 구현한다. 



### **Two’s complement division by a power of 2, rounding up**

- proper한 반올림을 하기 위해 bias를 더하여 음수에 대한 improper 반올림을 수정할 수 있으며, 이 때 bias는 $2^k-1$이다. 

![2_3_2 7](https://github.com/user-attachments/assets/11966102-36bd-4523-8688-fc08c5bc7438)
- **DERIVATION**
    - C 프로그램은 round를 $\lfloor x/y \rfloor$로 처리하기 때문에 $x/y<0$에 대해서 $\lceil x/y \rceil$이 되도록 적절한 bias를 더해야 한다.

    - $\lceil x/y \rceil$= $\lfloor {x+y-1}/y \rfloor$ 라는 성질을 이용하자. 
        - $x =qy+r$이라고 하면       
            $\lfloor {x+y-1}/y \rfloor$ = $\lfloor {(q+1)*y+r-1}/y \rfloor$  = $q+1+\lfloor {r-1}/y \rfloor$ 
        - $0≤r<y$ 이므로         
            $\lfloor {x+y-1}/y \rfloor=q+1-1 = q$ (r=0)
            $\lfloor {x+y-1}/y \rfloor= q+1$    (r>0) 
        - 따라서 x/y<0 일 때 bias를 더하여  $\lceil x/y \rceil$가 되며 bias는 $y-1$ = $2^k-1$이다. 

- 위의 C expression은 
    
    $(x<0 ? x+(1<<k)-1 : x) >>k$
