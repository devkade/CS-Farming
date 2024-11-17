---
created: 2024-10-01T20:31
modified: 2024-11-17T17:44
---
# Map of Contents

- [CA_3.1_Historical_Perspective](./CA_3.1_Historical_Perspective.md)
- [CA_3.2_Program_Encodings](./CA_3.2_Program_Encodings.md)
- [CA_3.3_Data_Formats](./CA_3.3_Data_Formats.md)
- [CA_3.4_Accessing_Information](./CA_3.4_Accessing_Information.md)
- [CA_3.5_Arithmetic_and_Logical_Operation](./CA_3.5_Arithmetic_and_Logical_Operation.md)

# Learning Objectives


> [!NOTE] 
> - 컴퓨터는 바이트의 나열인 기계 코드를 실행한다.
> - 컴파일러는 운영체제에 따르는 규칙과 프로그래밍 언어에 따라 일련의 단계를 통하여 기계 코드를 생성한다.
> - C 컴파일러는 기계 코드를 어셈블리 코드로 생성하고, 어셈블러, 링커를 통해 실행 가능한 기계 코드를 생성한다.
> - 해당 장에서는 기계 코드와 기계 코드를 인간이 읽을 수 있도록 하는 어셈블리 코드에 대해 탐구한다.

## Why do we need to learn machine code?

- C, Java 와 같은 고급언어로 프로그래밍하고 최신 최적화 컴파일러를 사용하는 경우, 우리는 숙련된 어셈블리 개발자가 개발한 것과 같이 효율적으로 코드를 작성할 수 있다.

- 그렇다면 왜 기계 코드를 배워야 할까?
	- 어셈블리 코드를 생성하는 일을 컴파일러가 대부분의 작업을 하지만, 어셈블리 코드를 읽고 해석하는 능력은 중요하다.
	- 어셈블리 읽음으로써 1) 컴파일러의 최적화 기능을 이해하고, 2) 코드 내의 근본적인 비효율성을 분석할 수 있다.
	- 코드 성능을 극대화하려는 경우, 소스 코드를 여러 번 변형하고 어셈블리 코드를 분석함으로써 프로그램이 얼마나 효율적으로 실행될 지를 이해할 수 있다.

## Learning List

> [!Caution]
> - 해당 도서는 x86-64를 기초로 한다.

- C, 어셈블리 코드, 기계 코드 간의 관계
- 데이터 표현 및 조작, 제어의 구현
- if, while, switch 문 같은 C의 제어 구조의 구현
- Procedure의 구현(프로그램이 procedures 사이에서 데이터 전송과 조정을 지원하기 위해 런타임 스택을 어떻게 유지하는가)
- 배열, 구조체, 유니온과 같은 데이터 구조의 기계 수준에서 구현
- 메모리 참조 범위를 벗어난 경우, 버퍼 오버플로우에 관한 취약성 검토
- 부동 소수점 데이터 및 연산과 관련된 코드의 기계 프로그램 표현