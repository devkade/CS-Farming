---
created: 2024-11-10T22:21
modified: 2024-11-17T17:44
---
# Loops

- C는 여러 가지 루프를 제공한다. do-while, while, for
- 기계어는 대응하는 명령어가 존재하지 않는다. 때문에 조건과 점프의 조합이 루프를 구현하는데 사용된다.

## Do-While

```c
do
    body-statement;
while (test-expr);
```

```c
loop:
    body-statement;
t = test-expr;
if (t)
    goto loop;
```

- do-while 문은 다음과 같이 goto 문으로 표현 될 수 있다.
- 매 반복 마다 프로그램은 body-statement를 평가한 다음 테스트 표현식을 평가 한다.
- 테스트가 성공하면 다음 반복으로 돌아간다.


### 팩토리얼 연산 루프

```c
long fact_do(long n)
{
	long result = 1;
	do {
		result *= n;
		n = n-1;
	} while (n > 1);
	return result;
}
```

```c
long fact_do_goto(long n)
{
    long result = 1;
loop:
    result *= n;
    n = n - 1;
    if (n > 1)
        goto loop;
    return result;
}
```

```assembly
	# long fact_do(long n)
	# n in %rdi
	
fact_do:
	movl    $1, %eax            # Set result = 1
.L2: loop:
	imulq   %rdi, %rax          # Compute result *= n
	subq    $1,   %rdi          # Decrement n
	cmpq    $1,   %rdi          # Compare n:1
	jg      .L2                 # If >, goto loop
	rep; ret                    # Return
```

- jg   .L2 가 루프를 구현하는 핵심 명령어가 된다.
- 어셈블리 코드를 리버스 엔지니어링 하기 위해서는 레지스터가 어떤 값과 맵핑 되는지를 결정해야 한다.
- n 이 %rdi 로 전달되는 것을 알고 있다.
- %eax로 1이 전달된다.
	- %rax가 1로 초기화되는 것과 같다. (%rax의 상위 4바이트가 0으로 설정되기 때문)
- 이후 곱셈 연산은 %rax로 업데이트되고 %rax는 함수값을 반환하는데 사용되므로 %rax가 result에 해당한다고 결론 지을 수 있다.

---

## While

```c
while (test-expr) 
    body-statement
```

```c
goto test;
loop:
    body-statement
test:
    t = test-expr;
    if (t)
        goto loop;
```

- while은 do-while과 달리 test-expr가 평가되고 처음 body-statement를 실행하기 전에 종료될 수 있다.
- while을  기계 코드로 번역 하는 방법은 여러 가지가 있다. 2가지 방법이 gcc 에서 생성 된 코드 에서 사용된다.

### Jump to middle

```c
goto test;
loop:
    body-statement
test:
    t = test-expr;
    if (t)
        goto loop;
```

- 테스트를 위한 무조건 점프를 수행 하여 초기 테스트를 실행하고 만족할 경우 body-statement로 점프해 코드를 실행 한다.

#### 팩토리얼 연산 루프

```c
long fact_while_jm_goto(long n) {
    long result = 1;
    goto test;
loop:
    result *= n;
    n = n - 1;
test:
    if (n > 1)
        goto loop;
    return result;
}
```

```assembly
	# long fact_while(long n)
	# n in %rdi:
fact_while:
    movl   $1,   %rdi        # Set result = 1
	jmp    .L5               # Goto test
.L6:
	imulq  %rdi, %rax        # Compute result *= n
	subq   $1,   %rdi        # Decrement n
.L5:
    cmpq   $1,   %rdi        # Compare n:1
    jg     .L6               # If >, goto loop
    rep; ret                 # return
```

- 루프 전에 goto문을 통해 test로 이동해 test-expr를 먼저 테스트하도록 하였다.

### Guarded do

```c
t = test-expr;
if (!t)
    goto done;
do
    body-statement
while (test-expr);
done:
```

- while을 do-while로 변경하는 방식이다.
	- 루프 내부의 테스트 코드를 루프 앞에 초기 테스트로 1번 더 작성 해 주는 것이다.
- 초기 테스트가 실패하면 루프 건너 뛰도록 조건 브랜치를 사용한다.
- gcc는 더 높은 최적화 수준으로 컴파일 할 때 이 전략을 사용한다.
- 아래 같은 goto문으로 변환될 수 있다.

```c
t = test-expr;
if (!t)
    goto done;
loop:
	body-statement
	t = test-expr;
	if (t)
		goto loop;
done:
```

- 해당 구현 전략은 컴파일러가 첫 test-expr를 최적화 할 수 있는 경우가 많다.

#### 팩토리얼 연산 루프

- gcc에 -O1 컴파일을 하는 경우(더 높은 최적화를 하는 경우)

```c
long fact_while_gd_goto(long n) {
    long result = 1;
    if (n <= 1)
        goto done;
    loop:
        result *= n;
        n = n - 1;
        if (n != 1)
            goto loop;
    done:
        return result;
}
```

```
long fact_while(long n)
n in %rdi
fact_while:
    cmpq  $1, %rdi        # Compare n:1
	jle   .L7             # If <=, goto done
	movl  $1, %eax        # Set result = 1  
.L6:                   # loop:
    imulq %rdi, %rax      # Compute result *= n
    subq  $1, %rdi        # Decrement n
	cmpq  $1, %rdi        # Compare n:1
    jne   .L6             # If !=, goto loop
    rep; ret              # Return
 .L7:                  # done:
    movl $1, %eax         # Compute result = 1
    ret                   # Return
```

- 해당 코드의 경우 가장 첫 번째 조건에서 n <= 1인 경우 종료를 시킨다.
- 이전 코드와 비교했을 때 .L6의 루프 테스트가 n > 1 에서 n = 1로 변화한 점을 확인할 수 있다.
	- 이는 컴파일러가 n > 1 인 경우에만 루프에 들어갈 수 있고 n 은 감소하여 n = 1이 된다는 것을 판단했기 때문이다.

---

## For

```c
for (init-expr; test-expr; update-expr)
    body-statement
```

- C 언어 표준은 한가지 예외를 제외하고 for 루프의 동작이 다음의 while 루프를 사용하는 코드와 동일 하다고 명시 한다.

```c
init-expr;
while (test-expr) {
    body-statement;
    update-expr;
}
```

- 프로그램은 먼저 init-expr를 평가 한다.
- 이후 test-expr를 평가 하는 루프에 들어가며 테스트가 실패하면 종료 된다.
- 테스트가 성공하면 body-statement를 실행한다.
- 마지막으로 update-expr를 평가 한다.

- gcc가 for 루프를 위해 생성하는 코드는 최적화 수준에 따라 while 루프의 2가지 번역 전략(Jump to middle, Guarded-do) 중 하나를 따른다. 

### For with Jump to middle

```c
	init-expr;
	goto test;
loop: 
	body-statement;
	update-expr;
test:
	t = test-expr;
	if (t)
	    goto loop;
```

### For with Guarded-do

```c
	init-expr;
	t = test-expr;
	if (!t)
	    goto done;
loop: 
	body-statement;
	update-expr;
	t = test-expr;
	if (t)
	    goto loop;
done:
```

### 팩토리얼 함수 with For

```c
long fact_for(long n)
{
	long i;
	long result = 1;
	for (i = 2; i <= n; i++)
		result *= i;
	return result;
}
```

- For 문의 팩토리얼 함수 코드는 while, do-while 코드와는 다르다.
- 팩토리얼 함수의 for 루프 구성을 확인해보면 아래와 같다.
	- init-expr : i = 2
	- test-expr : i <= n
	- update-expr : i++
	- body-statement : result \*= i;

#### For to while

- 구성을 통해서 while문으로 변경해보자.(위의 변환 코드 참조)

```c
long fact_for_while(long n)
{
    long i = 2;
    long result = 1;
    while (i <= n) {
        result *= i;
        i++;
    }
    return result;
}
```

#### While to goto

- Jump to middle 을 통해서 goto로 변경해보자.

```c
long fact_for_jm_goto(long n)
{
    long i = 2;
    long result = 1;
    goto test;
loop:
    result *= i;
    i++;
test:
    if (i <= n)
        goto loop;
    return result;
}
```

#### Assembly Code of For Loop

- 실제로 gcc를 통해 -Og 옵션으로 어셈블리 코드를 생성하면 위의 과정을 통해 변환한 템플릿을 따른다.
- -Og : 기본적인 최적화 (컴파일된 기계 코드가 원래 소스 코드와 가깝게 유지되도록 생성)

```assembly
	# long fact_for(long n)
	# n in %rdi
	
fact_for:
	movl $1, %eax         # Set result = 1
	movl $2, %edx         # Set i = 2
	jmp .L8               # Goto test
.L9:                    # loop:
	imulq %rdx, %rax      # Compute result *= i
	addq $1, %rdx         # Increment i
.L8:                    # test:
	cmpq %rdi, %rdx       # Compare i:n
	jle .L9               # If <=, goto loop
	rep; ret              # Return
```

- 결과적으로 루프에 대한 세계의 형태는 비슷한 전략으로 변환 될 수 있고 하나 혹은 그 이상의 조건 브랜치들로 코드를 생성 할 수 있다.
- Conditional control transfer는 루프를 기계코드로 변경하는데 기본적인 구조를 제공한다.

