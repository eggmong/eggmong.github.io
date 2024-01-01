---
title:  "언리얼 메모 1"
expert: "Super 키워드, TEXT 매크로, 라이브 코딩 단축키, FString"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: true
toc_sticky: true
 
date: 2024-01-01
last_modified_at: 2024-01-01
---

<br>

# 상속받은 함수 호출 : Super

```cpp
Super::Init();
```

# 복잡한 문자열 처리를 하나로

- 유니코드를 사용해 문자열 처리를 통일
  - 2byte로 크기가 균일한 UTF-16을 사용
  - 유니코드를 위한 언리얼 표준 캐릭터 타입 : `TCHAR`
- 문자열은 언제나 TEXT 매크로를 사용해 지정
  - TEXT 매크로로 감싼 문자열은 TCHAR 배열로 지정됨
- 문자열을 다루는 클래스로 `FString` 제공
  - FString 은 TCHAR 배열을 포함하는 헬퍼 클래스

```cpp
void UMyGameInstance::Init()
{
	Super::Init();

	TCHAR LogCharArray[]  = TEXT("Hello Unreal");
	UE_LOG(LogTemp, Log, LogCharArray);

	FString LogCharString = LogCharArray;
	UE_LOG(LogTemp, Log, TEXT("%s"), *LogCharString);
}
```

> 💡 LogCharString 에 포인터를 붙이는 이유  
> UE_LOG 에서 세번째 구문에는 항상 배열만 들어가기 때문에  `FString`을 쓸 수 없음  
> 그래서 `%s` 로 포맷 써주고,  
> %s 에 대응될 때는 TCHAR 의 포인터 어레이를 반환해 줘야 하는데  
> <b>FString 을 그대로 쓰면 포인터 반환이 안 됨</b>. 그래서 포인터를 붙여줘야 함.  
> 포인터를 붙이면 FString 이 감싸고 있는 실제 문자열 데이터를 가져올 수 있게 됨  