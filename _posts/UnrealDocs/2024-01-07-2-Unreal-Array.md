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

[TArray 공식 문서 링크](https://docs.unrealengine.com/4.26/ko/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/TArrays/)


```cpp
// GameInstance 를 상속 받은 MyGameInstance

#include "MyGameInstance.h"
#include "Algo/Accumulate.h"

void UMyGameInstance::Init()
{
	Super::Init();

	const int32 ArrayNum = 10;
	TArray<int32> Int32Array;

	for (int32 ix = 1; ix <= ArrayNum; ++ix)
	{
		Int32Array.Add(ix);
	}

	Int32Array.RemoveAll(
		[](int32 Val)				// 람다 함수로 '짝수만 제거' 조건 추가
		{
			return Val % 2 == 0;
		}
	);

	Int32Array += {2, 4, 6, 8, 10};	// 다시 짝수 넣어주기

	TArray<int32> Int32ArrayCompare;
	int32 CArray[] = { 1, 3, 5, 7, 9, 2, 4, 6, 8, 10 };
	Int32ArrayCompare.AddUninitialized(ArrayNum);		// 배열 크기 지정
	FMemory::Memcpy(Int32ArrayCompare.GetData(), CArray, sizeof(int32) * ArrayNum);
	// 메모리 카피로 배열 복사해서 넣어주기

	ensure(Int32Array == Int32ArrayCompare);

	int32 Sum = 0;
	for (const int32& Int32Elem : Int32Array)
	{
		Sum += Int32Elem;
	}

	ensure(Sum == 55);

	int32 SumByAlgo = Algo::Accumulate(Int32Array, 0);		// #include "Algo/Accumulate.h"
	ensure(Sum == SumByAlgo);
}
```

- 람다 함수로 조건 추가 가능
- Algo/Accumulate.h 헤더 추가로 알고리즘 사용 가능



# 2. TSet

[STL 의 Set 링크](https://eggmong.github.io/cpp/1-STL-Map/#6-%EC%85%8B-set)

- 언리얼 TSet 특징
  - <b>해시테이블</b> 형태로 구축되어 있어 빠른 검색 가능
  - 동적 배열의 형태로 데이터가 모여 있음
  - 데이터를 삭제해도 재구축이 일어나지 않음
  - 비어있는 데이터가 있을 수 있음

오히려 unordered_set과 유사하게 동작하지만 동일하진 않다.  
중복 없는 데이터 집합을 구축하는데 유용하게 사용 가능  

[TSet 공식 문서 링크](https://docs.unrealengine.com/4.27/ko/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/TSet/)


```cpp
TSet<int32> Int32Set;
	for (int32 ix = 1; ix <= ArrayNum; ++ix)
	{
		Int32Set.Add(ix);
	}

	Int32Set.Remove(2);
	Int32Set.Remove(4);
	Int32Set.Remove(6);
	Int32Set.Remove(8);
	Int32Set.Remove(10);
	Int32Set.Add(2);
	Int32Set.Add(4);
	Int32Set.Add(6);
	Int32Set.Add(8);
	Int32Set.Add(10);
```

출력해보면 1, 10, 3, 8, 5, 6, 7, 4, 9, 2 로 나옴.  
순서 없는 집합이라는 걸 명심.