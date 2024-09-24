---
up: "[[CA 1 A Tour of Computer Systems]]"
related:
  - "[[CA 1.4 Processors Read and Interpret Instructions Stored in Memory]]"
tags:
  - 📝/🌱️
aliases: 
cssclasses:
  - dashboard
created: 2024-09-22T21:13
modified: 2024-09-22T22:44
---
## 1.3 It Pays to UnderStand How Compilation Systems Work

- Compilation System의 작동 방식을 이해해야 하는 이유
1. 프로그램 성능 최적화
	- C 프로그램을 코딩할 때 좋은 코딩 결정을 내리기 위함.
	- 3장 : x86-64(최근 Linux, Macintosh, Window 기계어) 소개
	- 5장 : 컴파일러가 작업을 더 잘 수행할 수 있도록 돕는 간단한 변환으로 C 프로그램의 성능을 조정하는 방법 습득
2. Link 시간 오류 이해
	- 가장 혼란스러운 프로그래밍 오류 중 일부는 링크 프로그램의 동작과 관련되어 있다.
		- ex) 링크 프로그램이 참조를 해결할 수 없다고 보고하는 경우
	- 7장에서 배울 예정
3. 보안 구멍 피하기
	- 안전한 프로그래밍의 첫 단계는 프로그램 스택에 데이터와 제어 정보가 저장되는 방식의 결과를 이해하는 것임.
	- 3장 : 스택 규칙과 버퍼 오버플로우 취약점 습득