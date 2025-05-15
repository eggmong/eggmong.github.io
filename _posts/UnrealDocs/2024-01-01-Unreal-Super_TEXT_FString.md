---
title:  "[정리] 언리얼 메모 1 (Super 키워드, TEXT 매크로, 등...)"
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

# 환경 세팅

- 에디터 설치 시  
  `디버깅을 위한 편집기 기호` 도 설치

- 에디터에서 `Edit` -> `Editor Preference` 에서,  
  `General` 의 `Region & Language` 에서 언어 바꿀 수 있고,  
  `General` 의 `Source Code` 항목에서 비쥬얼 스튜디오 버전 선택할 수 있음.

- 에디터에서 `Edit` -> `Project Setting` 으로 간 뒤,  
  `General` 의 `Maps & Mode` 로 가서 `Game Instance` 를 바꿔줄 수 있음.  
  - 게임 인스턴스는 어플리케이션의 뼈대... 싱글톤으로 관리가 됨
  - 언리얼이 만든 기본 GameInstance가 아닌 직접 만든 클래스로 바꿔줄 수 있음

- 마찬가지로 에디터에서 `Edit` -> `Project Setting` 으로 간 뒤,  
  `General` 의 `Maps & Mode` 로 가서 기본 생성된 맵을 Clear 할 수 있음.  
  그럼 로딩 시간이 줄어듦.  

- 비쥬얼 스튜디오 실행 후  
  `프로젝트 및 솔루션` -> `도구` -> `옵션` ->  
  `오류로 인해 빌드가 종료될 때 항상 오류 목록 표시`  체크 해제



# 언리얼 코드 컴파일 방법

- <b>헤더 파일</b>에 변경이 발생하면  
  👉 에디터를 끄고 비쥬얼 스튜디오에서 컴파일 한다.

- <b>cpp 파일</b>에 변경이 발생하면  
  👉 에디터에서 라이브 코딩으로 컴파일 한다. `(Ctrl + Alt + F11)`

- 비쥬얼 스튜디오에서 수동으로 클래스를 추가하지 말 것.



# 상속받은 함수 호출 : Super

```cpp
Super::Init();
```

`Super` 키워드는 부모 클래스를 가리킨다.
C#에선 `base.` 로 접근했었는데.  



# 로그 : UE_LOG

첫번째 인자에 로그 카테고리,  
두번째 인자에 로그 출력 수준 (Log, Warning, Error),  
세번째 인자에 출력할 내용의 포맷 (%s, %d...)  
네번째 인자에 출력할 내용을 입력한다.  

`UE_LOG(LogTemp, Log, TEXT("%s"), TEXT("Hello Unreal"));`



# 복잡한 문자열 처리를 하나로 : FString

- 유니코드를 사용해 문자열 처리를 통일
  - 2byte로 크기가 균일한 UTF-16을 사용
  - 유니코드를 위한 언리얼 표준 캐릭터 타입 : `TCHAR`
- 문자열은 언제나 `TEXT` 매크로를 사용해 지정
  - TEXT("Hello Unreal")
  - TEXT 매크로로 감싼 문자열은, TCHAR 배열로 지정됨
- 문자열을 다루는 클래스로 `FString` 제공
  - FString 은 TCHAR 배열을 포함하는 헬퍼 클래스

```cpp
void UMyGameInstance::Init()
{
	Super::Init();

  // 배열의 값을 출력할 땐, 배열 이름만 넣는다.
	TCHAR LogCharArray[]  = TEXT("Hello Unreal");
	UE_LOG(LogTemp, Log, LogCharArray);

  // FString의 경우 *를 붙여 역참조 해줘야 함. 역참조해서, 가리키는 값 = TCHAR*
	FString LogCharString = LogCharArray;
	UE_LOG(LogTemp, Log, TEXT("%s"), *LogCharString);
}
```

> 💡 LogCharString 에 포인터를 붙이는 이유  
> UE_LOG 에서 세번째 구문에는 항상 배열만 들어가기 때문에  `FString`을 쓸 수 없음  
> 그래서 `%s` 로 포맷 써주고,  
> %s 에 대응될 때는 TCHAR 의 포인터 어레이를 반환해 줘야 하는데  
> <b>FString 을 그대로 쓰면 포인터 반환이 안 됨</b>. 그래서 포인터를 붙여줘야 함.  
> `포인터를 붙이면` FString 이 감싸고 있는 실제 문자열 데이터를 가져올 수 있게 됨  



# 언리얼에서 사용하는 데이터 타입

## int32

- C#의 경우 int 타입은 4바이트인 int32로 명확히 정의가 되어 있음
- Unreal은 int를 사용하지 않고 `int32` 를 사용함

## bool, uint8( unsigned byte, 1바이트(8비트) )

- 참/거짓 데이터를 네트워크로 전송할 때, 또는 디스크에 저장할 때  
  `헤더파일`에 `bool 변수를 사용하지 않고`,  
  바이트 정보를 사용함. 그게 `uint8`. 어차피 1바이트라서 ㄱㅊ

- bool은 크기가 명확하지 않음... 그래서 헤더에선 bool 대신 uint8 쓰자

- 대신 일반 uint8 과의 구분을 위해 b접두사 사용

- cpp 로직에선 자유롭게 bool 사용 가능

