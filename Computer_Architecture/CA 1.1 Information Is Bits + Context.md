---
up: "[[CA 1 A Tour of Computer Systems]]"
related:
  - "[[CA 1.2 Programs Are Translated by Other Programs into Different Forms]]"
tags:
  - 📝️/🌱️
aliases: 
cssclasses:
  - dashboard
created: 2024-09-22T17:03
modified: 2024-09-23T19:22
---
## 1.1 Information Is Bits + Context

```c
//
// hello.c
// hello.c
#include <stdio.h>

int main(){
	printf("hello world")
	return 0;
}
```

- 프로그램은 텍스트 파일로 저장한 소스코드로 실행된다.
- 소스코드는 ASCII 코드를 통해서 8비트 뭉치로 조직된 비트이다.
	- 즉, 가장 첫 문자인 `#` 은 ASCII 코드로 35 인데, 35를 나타내는 0과 1의 뭉치로 표현된다.

- 시스템 내의 모든 문자는 ==비트로 표현된다==는 점이 핵심이다.

- 우리가 맥락을 다르게 구분해서 보기 때문에 서로 다른 데이터 객체로 구분될 뿐, 모든 문자는 비트로 표현된다.


---
BEFO:::
NEXT::: 