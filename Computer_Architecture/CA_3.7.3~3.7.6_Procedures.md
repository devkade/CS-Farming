## **3.7.3  Data Transfer**

- procedure가 호출되었을 때 control을 procedure로 넘기고, procedure가 반환될 때 control을 다시 돌려주는 것 외에도(3.7.2), argument(data)를 전달하거나 반환값을 받을 수 있다.
    - procedure p가 procedure q를 호출 할 때, p의 코드가 먼저 인자를 적절한 레지스터에 복사해놔야 한다. 마찬가지로 q가 p로 반환되면 p의 코드는 레지스터 %rax에 저장된 반환값에 접근할 수 있어야 한다.
- 대부분의 이 동작은 레지스터에 의해 수행된다.
- x86-64에서는 최대 여섯 개의 정수형(정수와 포인터) 인자가 레지스터로 전달될 수 있다.
    - 레지스터가 할당되는 순서는 정해져 있다.
        
        ![%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2024-11-12_130208](https://github.com/user-attachments/assets/7f6163df-5d24-4000-ac3f-0c67d33f8fb9)
- 함수가 6개 이상의 정수형 인자를 가질 때, 6개까지는 레지스터, 다른 인자들은 스택으로 전달된다.
    - 인자 1~6은 적절한 레지스터에 복사하고, 인자 7에서 n까지는 인자 7을 스택 탑에 넣는 방법으로 저장한다. → 이부분이 Argument build area이다.
    - p가 q를 호출했을 때 q가 전달하는 인자의 개수가 6이 넘어가면 p의 스택프레임에 argument를 push하여 Argument build area에 저장된다.
        
       ![%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2024-11-19_181220](https://github.com/user-attachments/assets/19492d54-601e-47f5-86ce-f74453fd9c6e)
 

아래 예시는 서로 다른 크기(8, 4, 2, 1 바이트)의 정수와 다양한 포인터 타입(8 바이트)을 포함한 8개의 인자가 어떻게 저장되는지 보여준다. → 6개는 레지스터, 2개는 스택프레임

![%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2024-11-19_181235](https://github.com/user-attachments/assets/114acd2f-340c-446f-9ae1-80d5ad117623)

![%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2024-11-19_181239](https://github.com/user-attachments/assets/18f104da-f963-44d1-a2a4-ae007c206532)

## **3.7.4  Local Storage on the Stack**

- 지금까지 살펴본 예시는 모두 레지스터에 저장 가능한 데이터이기 때문에 local storage가 필요하지 않았지만 지역 데이터(local data)가 메모리에 저장되어야 하는 경우가 있다.
- 지역 데이터가 메모리에 저장되어야 하는 경우가 있다.
    1. 사용할 수 있는 레지스터가 부족하여 모든 지역 데이터를 저장할 수 없는 경우.
    2. 주소 연산자 ‘&’가 지역 변수에 적용되어 해당 변수의 주소를 생성해야 하는 경우.
    3. 지역 변수가 배열이나 구조체로 선언되어, 배열 또는 구조체 참조를 통해 접근해야 하는 경우.
        - 배열이나 구조체로 선언된 경우에는 메모리 배치 방식을 이후에 자세히 다룬다.
- 위 세가지의 경우에 sp를 감소시켜 스택 프레임에 공간을 할당하는데 이것이 “Local variables” 부분이다.

*** 

***Example***

다음은 주소 연산자를 처리하는 방식을 보여주는 예시로, swap_add 함수와 caller 함수이다.
- swap_add :  포인터 xp와 yp가 가리키는 두 값을 교환한 뒤, 두 값의 합을 반환
- caller : 로컬 변수 arg1과 arg2의 포인트를 생성하여 swap_add에 전달
![스크린샷 2024-11-19 181246](https://github.com/user-attachments/assets/0a5c2c1b-64f9-4f1b-b346-829956d3ced3)

- 3.31(b)는 caller가 이러함 로컬 변수를 구현하기 위해 스택 프레인을 사용하는 방법을 보여준다.
    - 스택 포인터를 16 감소시켜 공간을 할당하고, arg1와 arg2를 저장한다.
- swap_add 호출이 완료되면, caller 함수는 스택에서 두 값을 가져온다.(line 8,9)

*** 

***Example***

다음은 더 복잡한 call_proc 함수로, 로컬 변수를 스택에 저장해야 할 뿐만 아니라, 8개의 인자를 가진 proc(3.7.3) 함수를 호출하기 위해 값을 전달해야 한다.

![스크린샷 2024-11-19 181253](https://github.com/user-attachments/assets/4fb10e02-0aee-4894-9e8a-571e9879bf44)

![스크린샷 2024-11-19 181258](https://github.com/user-attachments/assets/12242a76-5cc5-402b-8611-35e80d6034a2)

- line 2~15는 proc 호출을 준비하는 데 사용된다. 
    - 로컬 변수와 함수 매개변수를 위한 스택 프레임 설정, 함수 인자를 레지스터에 로드하는 작업이다.
- 7, 8번째 인자인 x4와 &x4 만 스택 포인터에 저장되고, 나머지 인자는 레지스터에 저장됨

## **3.7.5 Local Storage in Registers**

- 프로그램 레지스터들은 모든 procedure들이 공유하는 단일 자원의 역할을 한다.
    - 특점 시점에 하나의 procedure만 활성 상태일 수 있다.
    - 한 procedure(호출자, caller)가 다른 procedure(피호출자, callee)를 호출할 때, 피호출자는 호출자가 나중에 사용할 레지스터 값을 덮어쓰지 않도록 해야한다.
1. **Callee-Saved 레지스터**
    - 레지스터 %rbx, %rbp, %r12–%r15는 callee-saved 레지스터로 분류된다. procedure P가 procedure Q를 호출할 때, Q는 이러한 레지스터의 값을 보존해야 한다.
    - Q가 레지스터 값을 보존하는 방법은:
        1. 레지스터 값 변경하지 않는다.
        2. 원래 값을 스택에 push한 후 레지스터를 변경하고, 반환 전에 스택에서 값을 pop
    - 이 때 스택에 push하여 저장된 레지스터 부분을 “Saved Registers”라고 한다.
2. **Caller-Saved 레지스터**
    - 스택 포인터 %rsp를 제외한 모든 레지스터는 caller-saved 레지스터로 분류된다. 이는 레지스터가 어떤 함수에 의해서든 자유롭게 변경될 수 있음을 의미한다.
    - Q는 이 레지스터를 자유롭게 변경가능, 호출자 P는 Q를 호출하기 전에 해당 데이터를 먼저 저장해놔야 한다.

**동작차이**

- caller-saved 는 P가 호출 전에 스택에 저장
- callee-saved 는 Q가 변경하지 않거나, Q가 호출되었을 때 레지스터를 저장해놓고 사용하다가 반환할 때 pop

***Example***

![%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2024-11-19_181305](https://github.com/user-attachments/assets/ce8ad300-d013-459f-8cb5-89d895618554)

- 위 예제는 procedure p가 procedure Q를 두 번 호출
1. 첫 번째 호출 시, P는 값 x를 나중에 사용해야 하기 때문에 저장해야 함 (line 5)
2. 두 번째 호출 시, Q(y)의 계산된 값을 저장해야 함(line 8)
    - 위의 어셈블리 코드를 보면 %rbx, %rbp 두 개의 callee-saved 레지스터를 사용하는 것을 볼 수 있다.
    - %rbx : Q(y)의 계산된 값을 저장
    - %rbp: 값 x를 저장
- 반환 후 두 개의 callee-saved 레지스터 값을 스택에서 pop(line 13,14)
- 이 때 pop은 push의 역순으로 수행됨
    - 스택이 후입선출(LIFO)를 따르기 때문

## **3.7.6 Recursive Procedures**

- 지금까지의 방법으로 x86-64 procedure는 재귀적으로 자기 자신을 호출할 수 있다.
    - 각 procedure 호출은 스택에서 자신의 독립적인 공간을 가지므로, 동시에 실행 중인 여러 호출의 지역 변수들이 서로 간섭할 수 없다.
    - 스택 규칙은 procedure가 호출될 때 지역 저장 공간을 할당하고, 반환하기 전에 이를 해제하는 적절한 정책을 자연스럽게 제공

***Example***

![%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2024-11-19_181312](https://github.com/user-attachments/assets/35858e8a-150a-440b-8f2b-63e9fc0e56f8)

- %rbx는 %rdi에 있던 매개변수 n을 저장하고 스택에 저장됨(callee-saved register) (line 2)
- n의 값은 계산 후 반환되기 전에 스택에서 복원됨(line 11)
- 스택 규칙과 레지스터 저장 규칙에 의해, 재귀 호출 rfact(n-1)이 반환될 때(line 9)
    - 호출 결과는 %rax레지스터에 저장
    - 매개변수 n의 값은 여전히 %rbx 레지스터에 유지
    - 위 두 값을 곱하여 원하는 n*rfact(n-1)을 얻는다.