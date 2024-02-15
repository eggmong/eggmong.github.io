---
title:  "언리얼 메모 2 / FString 정리 추가"
expert: "FString"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: true
toc_sticky: true
 
date: 2024-01-03
last_modified_at: 2024-01-03
---

<br>

# FString 정리 추가



`FString`으로 받은 문자열을 끄집어내려면 포인터 연산자를 사용한다.  
그리고 끄집어내면 TCHAR 포인터 로 받게 된다.

## FString의 TArray 가져오고, 포인터 가져오기

```cpp
TCHAR LogCharArray[]  = TEXT("Hello Unreal");
UE_LOG(LogTemp, Log, LogCharArray);

FString LogCharString = LogCharArray;
UE_LOG(LogTemp, Log, TEXT("%s"), *LogCharString);

const TCHAR* LongCharPtr = *LogCharString;          // 바꿀 수 없도록 보통 const로 받음

// 근데 저 const가 아닌 포인터를 받아서 뭔가 고치고 싶다면
// FString이 관리하고 있는 TArray 의 첫번째 인자인 포인터를 가져와야 함
TCHAR* LogCharDataPtr = LogCharString.GetCharArray().GetData();
// TArray를 가져온 후, GetData()로 포인터 가져올 수 있음

TCHAR LogCharArrayWithSize[100];
FCString::Strcpy(LogCharArrayWithSize, LogCharString.Len(), *LogCharString);
// 저수준 복사
```



## Contains 으로 문자열 검색

```cpp
if (LogCharString.Contains(TEXT("unreal"), ESearchCase::IgnoreCase))    // IgnoreCase : 대소문자 구분 X
{
    int32 Index = LogCharString.Find(TEXT("unreal"), ESearchCase::IgnoreCase);
    // "unreal" 이 위치한 인덱스 가져올 수 있음
}
```



## 문자열 자르기 Mid, Split

```cpp
FString EndString = LogCharString.Mid(Index);
// "unreal" 이 시작되는 위치에서부터 끝까지 잘라줄 수 있음

UE_LOG(LogTemp, Log, TEXT("Find Test : %s"), *EndString);   // 포인터 연산자 항상 넣어주기! FString!
// 출력 결과 : Unreal (Hello Unreal 에서 unreal 찾고 거기서부터 끝까지 자른거니까)

FString Left, Right;
if (LogCharString.Split(TEXT(" "), &Left, &Right))
{
    UE_LOG(LogTemp, Log, TEXT("Split Test : %s 와 %s"), *Left, *Right);
    // 출력 결과: Split Test : Hello (이상한문자) Unreal
}
```

> 🤯 이유 : 윈도우에서 한글을 썼는데, CP949 형태에 멀티 바이트 스트링으로 저장이 되어서
> UTF-16을 쓰는 언리얼과 호환이 안되는 것.
> 💡 해결 방법: 비쥬얼 스튜디오에서 Save As를 누른 후, 인코딩 옵션에서 UTF-8 선택하여 저장
> 재컴파일 하면 잘 출력 됨



## int32, float을 FString으로 변환

```cpp
int32 IntValue = 32;
float FloatValue = 3.141592;

FString FloatIntString = FString::Printf(TEXT("Int:%d FLoat:%f"), IntValue, FloatValue);
FString FloatString = FString::SanitizeFloat(FloatValue);
FString IntString = FString::FromInt(IntValue);

UE_LOG(LogTemp, Log, TEXT("%s"), *FloatIntString);      // 출력 결과: Int:32 FLoat:3.141592
UE_LOG(LogTemp, Log, TEXT("Int:%s Float:%s"), *IntString, *FloatString);
// 출력 결과: Int:32 FLoat:3.141592
```


## FloatString을 int로 변환

```cpp
int32 IntValueFromString = FCString::Atoi(*IntString);
float FloatValueFromString = FCString::Atof(*FloatString);
FString FloatIntString2 = FString::Printf(TEXT("Int:%d Float:%f"), IntValueFromString, FloatValueFromString);
// 출력 결과: Int:32 FLoat:3.141592
```

# FString의 구조와 활용

![path]({{ site.imageurl }}Images/UnrealFString.png)  

- 다른 타입에서 FString으로의 변환
  - FString::Printf
  - FString::SanitizeFloat
  - FString::FromInt
- C런타임 수준에서 문자열을 처리하는 클래스 FCString
  - 문자열을 찾는 strstr
  - 문자열을 복제하는 Strcpy
- FString에서 다른 타입으로의 변환 (안전하진 않음)
  - FCString::Atoi
  - FCString::Atof