# 3.7 Procedures

**procedure는 중요한 abstraction 요소이다.**

- 특정 기능을 수행하는 code를 지정된 argument와 return값과 함께 packaging하는 방법을 제공
- 잘 설계된 소프트웨어는 procedure를 abstraction 메커니즘으로 사용한다.
    -  어떤 값이 계산될 것이며 procedure가 프로그램 상태에 어떤 영향을 미칠 것인지에 대한 명확하고 간결한 인터페이스 정의를 제공하면서 세부 구현은 숨긴다.

**procedure를 제공할 때 처리해야 하는 다양한 속성**

- procedure P가 procedure Q를 호출하고, Q가 실행을 마친 후 다시 P로 돌아가는 상황 가정
1. *passing control*  : pc(program counter)는 Q의 code 시작 주소로 설정, Q에서 반환 될 때는 P의 호출 후 명령어로 설정
2. *Passing data* : P는 Q에 하나 이상의 매개변수를 제공할 수 있어야 하며, Q는 P로 값을 반환할 수 있어야 한다.
3. *Allocating and deallocating memory*: Q는 시작 시 로컬 변수 공간을 할당하고 반환 전에 해당 저장 공간을 해제해야 할 수 있습니다.

## 3.7.1  The Run-Time Stack

- procedure 호출 메커니즘에 사용되는 기능이다.
- 후입선출(LIFO) 메모리 관리 방식을 사용한다.

**procedure P가 procedure Q를 호출하는 예시에서 P와 Q의 동작**

- Q가 실행 중일 때는 P까지의 모든 procedure가 일시적으로 중단.
- Q가 실행 중일 때, local 변수에 대한 새로운 저장 공간을 할당하거나 다른 procedure를 호출하는 데 필요한 설정은 반드시 Q가 수행한다.
- Q가 반환될 때 할당된 모든 local 저장 공간을 해제할 수 있다.
- P가 Q를 호출하면 제어 및 데이터 정보가 스택 끝에 추가되며, P가 반환될 때 이 정보는 해제된다.-> 즉 이러한 정보는 P frame에 존재

![스크린샷 2024-11-12 130208](https://github.com/user-attachments/assets/6f9cf881-4af8-4f42-ba6d-4a16dd1c2428)

**Stack Frame**
<br>

*stack frame이란 함수가 호출될 때마다 스택에 생성되는 데이터 구조로, 함수 실행에 필요한 여러 정보를 저장하는 공간이다.*

<br>

- x86-64의 stack은 작은 주소 방향으로 성장하며, stack pointer %rsp는 stack의 최상위 원소를 가리킨다.
- 데이터는 pushq와 popq instruction을 이용해서 stack에 저장되고 읽어올 수 있으며, 이 때 %rsp가 움직이며 공간 할당과 해제를 한다.
- x86-64 procedure가 register에 담을 수 있는 용량을 초과하는 저장 공간을 필요로 할 때, 스택에 공간을 할당.→ 이 영역을 procedure의 stack frame
    - register에는 최대 6개까지 저장 가능 (%rdi, %rsi, %rdx, %rcx, %r8, %r9)
- x86-64 procedure는 공간 및 시간 효율성을 위해 필요한 만큼의 stack frame을 할당
    - 지금까지 살펴본 함수들은 stack frame이 필요하지 않음(전달 인수가 6개를 넘지 않는다.)

- 위의 그림 3.25는 run time stack의 전체 구조를 보여주며, stack frame으로 분할된 가장 일반적인 형태
    - 현재 실행 중인 procedure의 frame은 항상 스택의 최상단에 위치(그림상 맨 아래)
    - procedure P가 Q를 호출하면 Q가 반환 될 때 돌아가야 할 return address를 P stack frame에 push
    - Q는 stack의 경계를 확장하여 Q의 stack frame에 필요한 공간 할당
        - saved register - register 값 저장
        - Local variables - local 변수에 대한 공간 할당
        - Argument build area - 호출하는 procedure argument 설정
    - 대부분의 procedure의 stack frame은 고정 크기이며 procedure 시작 시 할당됨
        - 일부는 가변 크기 frame → 3.10.5절

## 3.7.2 Control Transfer

- 함수 P에서 함수 Q로 제어를 전달하는 것은 단순히 PC를 Q 코드의 시작주소로 설정하는 것으로 이루어진다.
- 나중에 Q가 반환될 때, 프로세서 P의 실행을 다시 시작해야 하는 코드 위치의  기록을 갖고 있어야 한다.
    - 이 정보는 x86-64 머신에서 `call Q` instruction을 사용해 Q procedure를 호출함으로써 기록됩니다.
    - `call`
        - 주소 A를 스택에 푸시하고 PC를 Q의 시작으로 설정한다.
        - 푸시된 주소 A는 리턴주소라고 하며, call instruction 바로 다음 instruction의 주소로 계산된다.
        - 호출된 procedure가  시작하는 instruction의 주소를 목적지로 갖는다.
    - `ret`
        - call과 대응되는 instruction
        - ret는 주소 A를 스택에서 pop해오고 PC를 A로 세팅한다.

<br>

**multstore와 main 함수에서 call과 ret 명령어 실행 과정**
-  call instruction은 함수의 시작 부분으로 제어를 이동하는 반면(%rip(pc) = 0x400540), ret instruction은 call 다음에 오는 instruction으로 제어를 되돌린다.(%rip = 0x40056B)

![스크린샷 2024-11-12 130223](https://github.com/user-attachments/assets/0d35e891-dc56-441e-9a43-ca0a05f65d76)
![스크린샷 2024-11-12 130231](https://github.com/user-attachments/assets/d3c1f80f-fb97-40ee-9d88-dc9ec5d62619)


*과정*
---
- main의 주소 0x400563에 있는 call 명령이 multstore 호출.
    - call은 반환되고 난 후의 명령 주소 0x400568을 push하고 0x400540으로 점프
- retq가 실행될 때 까지 순차적으로 명령어 실행
- retq 실행
    - stack에서 0x400568을 pop하고 이주소로 점프하여 main함수 명령어 재개
---
<br>

**제어 전달의 더 상세한 예시.**

- top과 leaf 함수가 호출되는 디스어셈블 코드

![스크린샷 2024-11-12 130241](https://github.com/user-attachments/assets/1beb5d04-6660-408d-a2ba-043035ec2fa4)

*과정*
---
- main의 주소 0x40055b에 있는 명령어 callq 수행
    - 다음 명령 주소 0x400560 push하고 0x400545(Top)로 점프
- T1으로 점프 후 수행되다가 T2에서 callq 수행
    - 다음 명령 주소 0x40054e push하고 0x400540(Leaf)으로 점프
- L1으로 점프 후 수행되다가 retq 수행
    - *%rsp = 0x40054e pop하고 T3로 이동
- T3 수행하고 T4에서 retq 수행
    - 0x400560pop 하고 M2로 이동
---
→ %rdi  = x(변수), %rax = return 값, %rsp = stack pointer, *%rsp = sp안에 있는 값

**결론**
- 리턴 주소를 stack에 push하는 간단한 방법을 사용해서 함수가 나중에 프로그램의 적절한 위치로 리턴이 가능하게 된다.
- 스택이 제공하는 후입선출(LIFO) 메모리 관리 방식과 일치한다.