---
created: 2024-11-07T01:12
modified: 2024-11-11T00:06
---
# 3.6.6_Implementing_Conditional_Branches_with_Conditional_Moves

## Conditional Transfer. Control vs. Data

```c
if (a > b) {
    printf("A is greater than B");
} else {
    printf("B is greater than or equal to A");
}
```

- 제어를 조건에 전송하는 방식(Conditional Transfer of Control, A 방식, 조건부 제어)
	- Conditional Jump
	- 조건부 동작을 구현하는 전통적인 방식
	- 조건의 참 거짓에 따라 서로 다른 코드 경로로 프로그램에 실행 흐름이 변경 된다.
	- 간단하고 일반적이나, 현대 프로세서에서는 매우 비효율적일 수 있다.

```c
int result;
result = (a > b) ? 1 : 0; // 조건을 미리 평가하여 결과를 담음
if (result == 1) {
    printf("A is greater than B");
} else {
    printf("B is greater than or equal to A");
}
```

- 데이터를 조건에 전송하는 방식(Conditional Transfer of Data, B 방식, 조건부 이동)
	- Conditional Move
	- 조건 작업의 두 결과를 모두 계산한 다음 조건의 참 거짓 여부에 따라 하나를 선택하는 방식
	- 조건에 평가 결과에 관계 없이 두 결과를 계산하고 저장 하는 과정이 필요하나, 이후 실행 흐름에서 CPU는 조건을 평가하는데 따른 분기 예측을 걱정할 필요가 없다.
		- 멀티코어 연산 혹은 고속 연산을 수행할 수 있는 프로세서에서는 불필요한 분기 예측 실패가 조건을 계산하고 저장 하는 과정에 비해 더 높은 비용이 든다.
	- 현대 프로세서의 성능 특성에 더 잘 맞는다.

### Code 3.16

```c
long lt_cnt = 0;
long ge_cnt = 0;

long absdiff_se(long x, long y) {
    long result;
    if (x < y) {
        lt_cnt++;
        result = y - x;
    } else {
        ge_cnt++;
        result = x - y;
    }
    return result;
}
```

```c
long gotodiff_se(long x, long y) {
    long result;
    if (x >= y)
        goto x_ge_y;
    lt_cnt++;
    result = y - x;
    return result;
x_ge_y:
    ge_cnt++;
    result = x - y;
    return result;
}
```

```assembly
	# long absdiff_se(long x, long y)
	# x in %rdi, y in %rsi
	
absdiff_se:
	cmpq %rsi, %rdi           # Compare x:y
	jge .L2                   # If >= goto x_ge_y
	addq $1, lt_cnt(%rip)     # lt_cnt++
	movq %rsi, %rax
	subq %rdi, %rax           # result = y - x
	ret                       # Return
.L2:                          # x_ge_y:
	addq $1, ge_cnt(%rip)     # ge_cnt++
	movq %rdi, %rax
	subq %rsi, %rax           # result = x - y
	ret                       # Return
```

- if 조건에 따라서 분기가 갈라지는 형태를 갖는다.
- 제어를 조건에 전송하는 방식의 어셈블리 코드이다.


### Code 3.17

```c
long absdiff(long x, long y) {
    long result;
    if (x < y)
        result = y - x;
    else
        result = x - y;
    return result;
}
```

```c
long cmovdiff(long x, long y) {
    long rval = y - x;
    long eval = x - y;
    long ntest = x >= y;
    if (ntest) rval = eval;
    return rval;
}
```

```assembly
long absdiff(long x, long y)  # x는 %rdi, y는 %rsi에 저장됨
absdiff:
    movq %rsi, %rax          # rax에 y를 이동
    subq %rdi, %rax          # rval = y - x
    movq %rdi, %rdx          # rdx에 x를 이동
    subq %rsi, %rdx          # eval = x - y
    cmpq %rsi, %rdi          # x와 y 비교
    cmovge %rdx, %rax        # x >= y이면 rval = eval
    ret                       # rval 반환
```

- cmovge : Conditional Move Greater or Equal
- 데이터를 조건에 전송하는 방식의 어셈블리 코드이다.

### Difference between Control Transfer and Data Transfer

- 프로세서는 파이프라인을 통해 높은 성능을 달성한다.
	- 메모리에서 지시어 가져오기
	- 지시어 타입 결정하기
	- 산술 연산 수행 하기
	- 메모리 쓰기
	- 프로그램 카운터 업데이트하기
- 프로세서는 위의 각각의 단계들을 구속 단계와 겹치게 하여 높은 성능을 달성한다. 
	- e.g. 이전 지시어에 대한 산술 연산을 수행 하는 동안 하나의 지시어를 가져온다.
	- 지시어를 가져오기 위해서는 앞으로 실행할 지시의 순서를 미리 잘 결정 하여 파이프라인을 실행할 지시어를 결정해놔야 한다.
- 기계가 conditional jump(branch 선택)를 해야 하는 경우 브렌치 조건을 평가 할 때까지 어떤 방향에 프렌치로 이동 해야 할지 알 수 없다.
	- 이때 미리 이동 할 수 있게 하기 위해서 정교한 분기 예측 논리를 사용하여 지시어를 가득 채워 놓는다.
	- 현대 마이크로 프로세서 설계는 90% 정도의 성공률을 목표로 한다.

- 점프에 대해 잘못 예측 할 경우, 프로세서는 수행한 미래 지시어의 작업을 대부분 버려야 한다.
- 즉, 올바른 위치에서 다시 시작해 지시어로 파이프라인을 다시 채워야 한다.
- 잘못된 예측이 패널티를 발생 시킬 수 있고 프로그램의 성능에 저하를 초래 할 수 있다.

#### With Code 3.16, 3.17

- x < y 에 대한 전형적인 애플리케이션의 테스트 결과는 예측할 수 없고 50% 확률로만 추측 할 수 있다.
- 조건에 따라 실행 되는 두 코드는 다닐 클럭 사이클만 필요하기 때문에 분기 예측 실패의 패널티가 해당 함수의 성능을 좌우한다.
- x86-64 코드에 대해 실험해 본 결과
	- 브랜치 패턴이 쉽게 예측 될 때 함수 호출당 약 8 클럭 사이클이 소요
	- 브랜치 패턴이 무작위일 때는 호출당 약 17.5 클럭 사이클이 소요
	- 함수가 소요되는 시간은 브랜치의 예측이 맞는지 여부에 따라 약 8~27 사이클 사이에 분포

- 조건부 이동(B 방식), 데이터를 조건에 전달 하는 방식은 데이터에 흐름이 고정 되기 때문에 프로세스는 파이프라인을 더 쉽게 유지할 수 있다.
	- 때문에 분기 예측 실패 확률이 없으므로 언제나 약 8 클럭 사이클을 요구한다.

#### How to determine this penalty?

- $p$ : 잘못 예측 할 확률
- $T_{OK}$ : 잘못 예측 하지 않고 실행한 시간
- $T_{MP}$ : 잘못 예측한 패널티
- $T_{\text{avg}}(p)$ : $p$ 의 함수로 코드를 실행하는데 걸린 평균 시간

$$
T_{\text{avg}}(p) = (1 - p) T_{OK} + p (T_{OK} + T_{MP}) = T_{OK} + p T_{MP}
$$

- $T_{OK}$, $T_{ran}$ 과 $p = 0.5$ 일 때 평균 시간 $T_{MP}$ 결정 
- $T_{ran}$ : 랜덤하게 패턴이 호출될 경우 걸리는 시간

$$
\begin{gather}
T_{ran} = T_{\text{avg}}(0.5) = T_{OK} + 0.5 T_{MP} \\
T_{MP} = 2(T_{ran} - T_{OK}) \\
\end{gather}
$$
- $T_{OK}$ = 8, $T_{ran}$ = 17.5 일 때 
	- $T_{MP}$ = 19


### Assembly Conditional Movement

<img width="569" alt="image" src="https://github.com/user-attachments/assets/c531e6dd-cc3a-47f9-95e4-7ab8de3e4cba">

- x86-64에서 사용 가능한 일부 조건부 이동 명령을 나타낸다.
- S : 소스 레지스터
- R : 목적지 레지스터
- 소스 값은 메모리 또는 소스 레지스터의 읽어진다.
	- 지정된 조건이 충족 될 경우에만 목적지의 복사된다.
- 소스 및 목적지 값은 16, 32, 64 비트 길을 가질 수 있다.
	-  단일 바이트 조건부 이동은 지원 되지 않는다.
	- 바이트 길이는 목적지 레지스터의 이름에서부터 추측 할 수 있다.

- 조건부 이동은 프로세서가 테스트 결과를 예측할 필요 없이 조건부 이동 명령을 실행 할 수 있다.
- 프로세서는 단순히 소스 값을 읽고 조건 코드를 확인한 다음 목적지 레지스터를 업데이트하거나 동일하게 유지 한다.


## Conditional Operations

```c
v = test-expr ? then-expr : else-expr;
```

- 위 조건식을 방식에 따라 변경해보자.

### Conditional Control Transfer

```c
	if (!test-expr)
	    goto false;
	v = then-expr;
	goto done;
false:
	v = else-expr;
done:
```

- 이 코드는 두 개의 코드 시퀀스를 포함한다.
	- then-expr 를 평가
	- else-expr 를 평가
	- 즉, 둘 중 하나를 평가한다.


### Conditional Data Transfer

```c
v = then-expr;
ve = else-expr;
t = test-expr;
if (!t) v = ve;
```

- 이 코드는 then-expr, else-expr 모두 평가 되고 최종 값은 평가된 test-expr 를 기반으로 선택된다.
- 이 시퀀스의 마지막 문장은 조건부 이동으로 구현 된다.

- 모든 조건 표현식이 조건 이동을 사용하여 컴파일 될 수 있는 것은 아니다.
- 가장 중요한 것은 조건 이동을 사용한 추상 코드가 테스트 결과 관계 없이 then-expr, else-expr 모두를 평가 한다는 것이다.
	- 두 표현식 중 하나가 오류 조건을 생성하거나 부작용을 일으킬 가능성이 있다면 잘못된 동작으로 이어질 수 있다. 때문에 언제나 조건식 이동을 사용하는 것은 아니다.

#### Error with Conditional Move

```c
long cread(long *xp) {
    return (xp ? *xp : 0);
}
```

- 포인터가 null일 때 결과를 0으로 설정 하기 위해 조건 이동으로 컴파일 하기에 좋아 보인다.

```assembly
	# long cread(long *xp)
	# Invalid implementation of function cread
	# xp in register %rdi
	
cread:        
	movq  (%rdi), %rax             # v = *xp
	testq %rdi, %rdi               # Test x
	movl  $0, %edx                 # Set ve = 0
	cmove %rdx, %rax               # If x==0, v = ve
	ret                            # Return v
```

- 다음과 같이 어셈블리 코드로 나타냈을 때 이 구현은 유효하지 않다.
- movq (%rdi), %rax : 포인터 xp의 메모리 위치 값을 rax 레지스터로 이동시킨다.
	- 만약 xp 가 null이라면 잘못된 주소에 접근하므로 오류가 발생한다.
- 때문에 이 경우에는 브랜치 코드를 사용하여 컴파일 되어야 한다.

#### Low Code Efficiency with Conditional Move

- Conditional Move는 모든 모든 평가를 계산 한다.
- then-expr, else-expr 둘 중 하나라도 상당한 계산을 요구 한다면 조건이 충족 되지 않을 경우 낭비가 발생한다.

- 컴파일러는 낭비되는 계산에 상대적인 성능과 브랜치 예측 실패로 인한 패널티 가능성을 고려해야 한다.
- 하지만 사실 컴파일러는 고려를 통해 결정을 내리기에 충분한 정보를 갖고 있지 않는다.
- 때문에 실험을 해 봤을 때 gcc는 두 표현식이 매우 쉽게 계산 될 수 있을 때만 조건 이동을 사용한다.

## Conclusion

- Conditional Data Transfer 가 Conditional Control Transfer 에 대한 대한 전략으로 사용된다.
- Conditional Data Transfer 는 제한 된 경우에만 사용 되지만, 사용 되는 경우가 꽤 흔하고 현대 프로세서의 작동과 잘 일치한다.

