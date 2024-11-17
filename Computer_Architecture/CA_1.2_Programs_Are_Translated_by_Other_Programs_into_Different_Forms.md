---
up: "[[CA 1 A Tour of Computer Systems]]"
related:
  - "[[CA 1.3 It Pays to UnderStand How Compilation Systems Work]]"
tags:
  - 📝/🌱️
aliases: 
cssclasses:
  - dashboard
created: 2024-09-22T21:12
modified: 2024-11-17T17:44
---

## 1.2 Programs Are Translated by Other Programs into Different Forms

![alt text](<../Assets/Computer_Architecture/CA 1. A Tour of Computer Systems-1726993752388.jpeg>)

#### Preprocessing phase

hello.c 소스 프로그램(text)은 전처리기를 통해서 `#include <stdio.h>`와 같이 추가적으로 필요한 소스 코드를 추가하거나 수정되어 hello.i 소스 프로그램(text)으로 변환된다.

#### Compilation phase

hello.i 소스 프로그램(text)은 컴파일러를 통해 읽을 수 있는 hello.s 어셈블리 코드(text)로 변환된다.

```assambly
main:
  subq $8, %rsp
  movl $.LC0, %edi
  call puts
  movl $0, %eax
  addq $8, %rsp
  ret
```

#### Assembly phase

어셈블러는 hello.s 어셈블리 코드(text)를 기계어 명령으로 변환하고 hello.o 재배치가능한 객체 프로그램(binary)으로 변환된다.

> [!NOTE] 재배치가능한 객체 프로그램(Relocatable Object Program)
>
> -   컴파일된 코드의 이진 형태이다.
> -   메모리의 특정 주소에 고정되지 않고, 다양한 메모리 주소에 배치되어 실행할 수 있다.
>     -   주로 라이브러리, 모듈형 프로그램에서 이식성과 재사용성을 높이기 위해 사용된다.

#### Linking phase

hello 프로그램은 printf 함수를 호출하는데, printf 함수는 이전에 미리 컴파일된 객체 파일인 printf.o에 있고, printf.o와 hello.o를 연결해 printf 함수를 사용할 수 있도록 만든다. Linking을 통해 만들어낸 파일은 hello 라는 이름의 실행 가능한 객체 프로그램(binary)으로 변환된다.
