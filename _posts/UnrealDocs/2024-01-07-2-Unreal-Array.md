---
title:  "언리얼 컨테이너 라이브러리 / TArray, TSet"
expert: "TArray, TSet"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: true
toc_sticky: true
 
date: 2024-01-07
last_modified_at: 2024-01-07
---

<br>


# 1. TArray

- 가변 배열 자료구조
- STL 의 vector 와 동작 원리 유사
- 장점
  - 데이터가 순차적으로 모여있기 때문에 메모리를 효과적으로 사용할 수 있고, 캐시 효율 높음
  - 임의 데이터의 접근이 빠르고, 고속으로 요소를 순회하는 것이 가능
- 단점
  - 맨 끝에 데이터를 추가하는 건 가볍지만, 중간에 요소를 삽입하거나 삭제하는 건 작업 비용이 큼
- 데이터가 많아질 수록 검색, 삭제, 수정 작업이 느려지기 때문에  
  많은 수의 데이터에서 검색 작업이 빈번하게 일어난다면, `TSet`을 사용할 것.

![TArray](https://drive.google.com/uc?export=view&id=1ybStqniRgzX_gHVu4CVYLIPpNPo_pR26)  

- 배열을 시작하는 부분의 포인터를 `GetData()` 함수로 가져올 수 있음.
- Insert, Remove 는 중간에 작업하는 거라 비용이 많이 듦
- 동일한 사이즈가 차곡차곡 쌓여있는 구조라 [](인덱스) 접근으로는 빠르게 접근 가능
- 