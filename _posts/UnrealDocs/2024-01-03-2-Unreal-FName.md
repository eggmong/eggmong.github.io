---
title:  "[정리] 언리얼 메모 3 / FName, FText"
expert: "FName, FText"

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


# 언리얼이 제공하는 다양한 문자열 처리

## FName

- FName : 에셋 관리를 위해 사용되는 문자열 체계
  - 에셋을 빠르게 찾고 싶을 때, 이름을 지정해주는 게 명확하지만,  
  string으로 저장하게 되면 찾고 비교하는 데에 오래 걸림.
  - <b>대소문자 구분 없음</b>
  - 한번 선언되면 바꿀 수 없음
  - 가볍고 빠름 (key-value 쌍)
  - 문자를 표현하는 용도가 아닌 <b> 에셋 키</b> 를 지정하는 용도로 사용.  
  빌드 시 <b>해시 값</b> 으로 변환됨.
  - 문자열을 찾기 위한 클래스지, 처리하기 위한 클래스가 아님요
    - FName과 글로벌 Pool
      - 문자열이 들어오면 해시 값을 추출해 키를 생성하여 FName 에 보관
      - FName 값에 저장된 값을 사용해 전역 Pool 에서 원하는 자료 검색해 반환
    - FName의 형성
      - 생성자에 문자열 정보를 넣으면 풀을 조사해 적당한 키로 변환
      - Find or Add
  - [FName 문서 링크](https://docs.unrealengine.com/4.27/ko/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/StringHandling/FName/)

![FName]({{ site.imageurl }}Images/UnrealFName.png)  

```cpp
FName key1(TEXT("PELVIS"));
FName key2(TEXT("pelvis"));
UE_LOG(LogTemp, Log, TEXT("FName 비교 결과 : %s"), key1 == key2 ? TEXT("같음") : TEXT("다름"));
```
출력 결과 : 같음

```cpp
for (int i = 0; i < 10000; ++i)
{
    FName SearchInNamePool = FName(TEXT("pelvis"));
}
```

이렇게 FName 생성자에 문자열을 넣으면  
FName이 하는 일은 문자열을 키로 변환한 다음에, 그 key가 전역 pool 에 있는지  
조사하는 작업을 거치게 되는데,  
for 문에서 계속 실행 되므로 오버헤드가 발생할 수 있음.  

```cpp
for (int i = 0; i < 10000; ++i)
{
    FName SearchInNamePool = FName(TEXT("pelvis"));

    const static FName StaticOnlyOnce(TEXT("pelvis"));
}
```

이렇게 짜면 처음 초기화 할 때 데이터를 저장해주고,  
local static 으로 선언했으니까 그 다음부턴 이걸 찾을 일이 없음.  



## FText

- FText : 다국어 지원을 위한 문자열 관리 체계
  - 일종의 `키` 로 작용함.
  - 별도의 문자열 테이블 정보가 추가로 요구됨.
  - 게임 빌드 시 자동으로 다양한 국가별 언어로 변환됨.
  - [FText 문서 링크](https://docs.unrealengine.com/4.27/ko/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/StringHandling/FText/)

> FString -> FName or FText  
> (FString: 문자열 변환 주체)

