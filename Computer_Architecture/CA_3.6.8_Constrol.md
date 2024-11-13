# 3.6.8 Switch Statements

- Switch문은 정수 인덱스 값에 따라 다중분기(if-else문 여러개)를 효율적으로 처리하는 기능을 제공한다.
- C 코드를 읽기 쉽게 해줄 뿐만 아니라 점프 테이블이라는 자료구조를 사용해서 효율적인 구현을 가능하게 한다.
- jump table
    - jump table은 배열이며, 배열의 각 항목은 해당 인덱스가 일치할 때 프로그램이 실행해야 할 코드의 주소를 저장. → 8byte
    - .rodata에 존재하는 table
    - switch문을 사용하면 컴파일러(gcc)가 정적 메모리 영역에 존재하는 rodata에 table을 생성한다.
- switch문을 실행하는 데 걸리는 시간이 case의 수에 독립적이라는 점이 장점이다.
- jump table은 여러 개의 case가 있을 때 사용하고, 값이 좁은 범위에 걸쳐 있는 경우에 사용된다.

<br>

  
   ### Switch문의 간단한 예시


1. switch_eg(3.22(a))는 Switch문의 c 코드이고, 3.23은 3.22(a)의 어셈블리 코드
2. switch_eg_impl(3.22(b))는  이 어셈블리 코드의 동작을 c로 표현한 것

![스크린샷 2024-11-12 130143](https://github.com/user-attachments/assets/d85c7207-09dd-41b8-899b-c4b2db2c3e74)

1. switch_eg 3.22(a)
    - switch 문은 여러 특징을 가진다.
        - 101, 105 - 비연속 범위를 가지고 label이 없는 case
            
            →default 값으로 처리함
            
            →비연속 범위 - do not span a contiguous range 
            
            - 정확한 뜻을 모르겠음
            - label이 없다는 것이 중요한 것 같음
        - 104,106 - 다중 label의 case
            - 똑같이 처리하는 case
        - 102 - break문이 없어 다른 case로 넘어가는 case
            - break없이 다음 case로 넘어간다.
2.  switch_eg_impl 3.22(b)
    - 배열 jt는 7개의 entries를 갖고, 배열의 요소들은 각각의 code block의 주소이다.
    - 이 code block들은 label로 정의된다.(ex. loc_A)
    - gcc는 코드 위치의 포인터를 생성하는 새로운 연산자 &&를 추가했다.
        - && : gcc의 goto문에서 사용되는 code location의 포인터 생성 연산자
    - 원래의 c 코드 (switch_eg)에서는 100~106으로 고정된 값일 경우이지만, switch 변수 n은 임의의 정수일 수 있다. 따라서 컴파일러는 n에서 100을 빼서 범위를 0에서 6으로 이동시키고, 이를 index라는 새로운 변수로 정의한다.
        - 100~106 - 100 = 0~6
    - index를 unsigned value로 설정하면 분기의 경우의 수를 줄일 수 있다.
        - index<0인 값을 unsigned로 하면 index>0으로 통일됨
    - index>6이면 모두 default case로 처리한다.

<br>

![스크린샷 2024-11-12 130150](https://github.com/user-attachments/assets/6f465c9a-907c-4353-bca2-4d37c21233a8)

### switch_eg의 assembly code

- 3.22(b)의 goto *jt[index]와 5 line의 jmp 명령어가 유사하게 동작 수행
- %rsi = index
- jmp *  - 간접 점프
    - 직접 점프 : 특정 주소로 직접 점프
    - 간접 점프 : 메모리의 특정 주소에 저장된 값을 참조하여 점
- jmp *   .L4(,%rsi,8)
    - L4에서 rsi에 저장된 값에 8을 곱한 값만큼 이동한 주소에 간접 점프
    
![스크린샷 2024-11-12 130158](https://github.com/user-attachments/assets/dcd26d6c-5423-441f-b579-1836de422457)
    
- 정적 메모리 영역에 저장되어 있는 rodata의 jump table
- rodata 안에는 quad(8바이트)가 있는데 각 quad는 해당 어셈블리 코드 label(예 : .L3)과 연결된 명령어 주소로 지정된다.
- L4는 할당의 시작을 표시
- 이 label과 연결된 주소가 간접 점프의 기준이 된다.

결론: 단일 점프 테이블이 있으면 다중 분기(여러 개의 if-else문)를 효율적으로 구현가능하다.