---
up: 
related: 
tags: []
aliases: []
cssclasses:
  - dashboard
created: 2024-11-17T21:35
modified: 2024-11-17T22:42
---
# 3.10.5_Supporting_Variable-Size_Stack_Frames

- 다양한 함수의 기계 수준 코드를 살펴봤을 때 대부분 컴파일러가 스택 프레임을 위해 미리 할당할 공간 양을 결정할 수 있는 특성을 가지고 있었다.
- 하지만, 일부 함수는 가변적인 양의 로컬 저장소를 필요로 한다.
- alloca 함수를 호출하는 경우 함수가 스택에서 임의의 바이트 수의 저장공간을 할당할 수 있고, 코드가 가변 크기의 로컬 배열을 선언할 때도 가변적인 양의 로컬 저장소를 필요로 할 수 있다.

```c
long vframe(long n, long idx, long *q) {
    long i;
    long *p[n];
    p[0] = &i;
    for (i = 1; i < n; i++)
        p[i] = q;
    return *p[idx];
}
```

- 해당 코드는 가변 크기 배열을 포함하는 함수의 예시를 제공한다.
- local array p 를 선언하는데, p의 크기는 argument n에 의해 변한다. 즉, n 에 따라 필요한 스택 메모리의 양이 달라진다. → 컴파일러는 함수의 스택 프레임을 위해 얼마만큼의 공간을 할당해야 할 지 결정할 수 없다.
- 프로그램은 로컬 변수 i의 주소의 참조를 생성하기 때문에 i를 스택에 저장해야 한다.
- 프로그램은 실행 중 로컬 변수 i와 배열 p의 요소에 모두 접근할 수 있어야 하고, 반환 시 함수는 스택 프레임을 해제하고 반환 주소가 저장된 위치로 스택 포인터를 조정한다.

## Frame pointer

```assembly
	# long vframe(long n, long idx, long *q)
	# n in %rdi, idx in %rsi, q in %rdx
	# Only portions of code shown
vframe:
2	pushq %rbp                 # Save old %rbp
3	movq %rsp, %rbp            # Set frame pointer
4	subq $16, %rsp             # Allocate space for i (%rsp = s1)
5	leaq 22(,%rdi,8), %rax     
6	andq $-16, %rax            
7	subq %rax, %rsp            # Allocate space for array p (%rsp = s2)
8	leaq 7(%rsp), %rax
9	shrq $3, %rax
10	leaq 0(,%rax,8), %r8       # Set %r8 to &p[0]
11	movq %r8, %rcx             # Set %rcx to &p[0] (%rcx = p)
		# . . .
	# Code for initialization loop
	# i in %rax and on stack, n in %rdi, p in %rcx, q in %rdx
.L3: loop:
13	movq %rdx, (%rcx,%rax,8)   # Set p[i] to q
14	addq $1, %rax              # Increment i
15	movq %rax, -8(%rbp)        # Store on stack
.L2:
17	movq -8(%rbp), %rax        # Retrieve i from stack
18	cmpq %rdi, %rax            # Compare i:n
19	jl .L3                     # If <, goto loop
		# . . .
	# Code for function exit
20	leave                      # Restore %rbp and %rsp
21	ret
```

![](https://i.imgur.com/NNUIKUE.png)

- 위 vframe의 어셈블리 코드와 vframe 함수를 위한 스택 프레임 구조 그림이다.
- 코드
	- %rbp의 현재 값을 스택에 저장하고 %rbp가 자신의 주소를 저장한 위치를 가리키도록 설정한다.
	- 스택에 16바이트를 할당한다.(s1)
		- 첫 번째 8바이트는 지역 변수 i를 저장하는데 사용된다. (long 이므로 8바이트)
		- 두 번째 8바이트는 사용되지 않는다.
	- 배열 p의 공간을 할당한다. (5-11 line)(s2)
		- 8n + 22 를 %rax에 저장하는데, 22는 offset을 나타내고, 8n의 공간을 할당해 배열의 공간을 만든다.
		- $-16과 %rax 를 and 연산함으로써 %rax의 하위 4비트를 0000으로 만든다. 해당 과정은 주소가 16의 배수가 되도록 만들어 메모리를 정렬하는 효과를 준다. (해당 주소를 찾기 쉽게 만든다.)
		- 10-11 line 의 %r8, %rcx는 p의 인덱스와 p 배열의 시작 주소를 나타낸다.
	- 배열 요소 p\[i\] 가 q로 설정된다.(13 line)
	- 지역 변수 i 가 업데이트 되고(15 line), 읽혀진다(17 line).
	- leave 명령어는 프레임 포인터를 이전 값으로 복원한다.(전체 스택 프레임을 할당 해제한다.) 해당 명령어는 아래 두 명령어를 실행하는 것과 동일하다.
		- `movq %rbp, %rsp # Set stack pointer to beginning of frame`
		- `popq %rbp       # Restore saved %rbp and set stack ptr to end of caller's frame`

- 가변 스택 프레임을 관리하기 위해 x86-64 코드는 레지스터 %rbp 를 frame pointer로 사용한다. (base pointer 라고 부르기도 하는데, 이때문에 %rbp의 bp가 사용된다.)
- 코드를 확인해보면 %rbp의 이전 버전을 스택에 저장해야 한다는 것을 볼 수 있는데, 이는 callee-saved register이기 때문이다.
	- Callee-Saved Register 는 함수가 호출된 후에도 그 값이 유지되어야 하는 레지스터를 말한다.
- 함수 실행 중 %rbp가 저장한 프레임 포인터 값을 가리키도록 유지한다.
	- %rbp에 대해 상대적인 offset을 통해 i와 같은 고정 길이 로컬 변수에 참조한다.

- 이전 x86 코드에서는 모든 함수 호출에 프레임 포인터가 사용되었으나, x86-64 코드에서는 스택 프레임의 크기가 가변적일 경우에만 사용된다.
