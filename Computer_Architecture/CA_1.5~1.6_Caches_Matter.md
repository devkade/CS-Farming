---
up: "[[CA 1 A Tour of Computer Systems]]"
related: []
tags:
    - 📝/🌱️
aliases:
cssclasses:
    - dashboard
created: 2024-09-22T21:53
modified: 2024-09-23T00:32
---

## 1.5 Caches Matter

### 캐시의 필요성

![[CA 1 A Tour of Computer Systems-1727007901180.jpeg]]

- 이 간단한 예에서도 시스템은 정보를 옮기는데 많은 시간을 소비한다.
  - 디스크에 저장되어 있는 `hello`(코드 + 데이터)를 메인 메모리로 이동
  - 출력물을 메인 메모리에서 레지스터 파일을 거쳐 디스플레이로 이동
- 과도한 복사는 오버헤드이므로 복사 작업이 가능한 빠르게 실행하도록 만드는 것이 목표가 된다.

- 물리적인 이유로 큰 저장 장치는 작은 저장 장치보다 느리다.
- 더 빠른 장치는 느린 장치에 비해 비용이 더 비싸다.
  - 메인 메모리(수십억 바이트)는 저장 공간이 레지스터(수백 바이트)에 비해 크다.
  - 크기로 인해 프로세서가 읽는 속도가 메인 메모리보다 레지스터를 읽는 것이 더 빠르다.
  - 메인 메모리를 개발해 접근 속도를 빠르게 만드는 것보다 프로세서를 더 빠르게 만드는 것이 더 쉽고 비용적으로 이득이다.
  - 때문에 프로세서는 더 빨리 읽을 수 있는 작은 저장소를 좋아하고, 프로세서에 맞춰 고속 저장소를 추가해준다.

### 캐시 메모리

![[CA 1 A Tour of Computer Systems-1727008973325.jpeg]]

- 캐시 메모리(Cache Memory)

  - 프로세서가 가까운 미래에 필요할 가능성이 있는 정보를 임시로 저장할 수 있는 공간
  - SRAM(Static Random Access Memory)를 통해 구현
  - 시스템이 특정 영역의 데이터와 코드에 반복적으로 접근하는 지역성을 이용해 큰 메모리와 빠른 메모리의 효과를 모두 얻을 수 있다는 아이디어로부터 개발됨.

- L1 Cache : 수만 바이트를 저장하여 레지스터 파일만큼 빠르게 접근 가능하다.
- L2 Cache : 수십만 ~ 수백만 바이트 저장. 효율성과 성능을 위해 프로세서와 특별한 버스를 통해 연결되어 있다.

  - L1 캐시보다 L2 캐시 접근하는 것이 5배 더 걸리지만, 메인 메모리에 비하면 5~10배 더 빠르다.

- 모든 컴퓨터 시스템의 저장 장치는 위와 같이 유사한 메모리 계층으로 조직되어 있다.
- 아래로 갈수록 느리고, 커지며, 바이트 당 비용이 저렴해진다.
- 메모리 계층의 주 아이디어
  - 한 레벨의 저장소가 다음 하위 레벨의 저장소를 위한 캐시 역할을 한다.
