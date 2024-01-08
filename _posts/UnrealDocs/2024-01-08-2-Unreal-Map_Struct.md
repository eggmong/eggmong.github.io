---
title:  "언리얼 컨테이너 라이브러리 / 구조체, Map"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: true
toc_sticky: true
 
date: 2024-01-08
last_modified_at: 2024-01-08
---

<br>


# 1. 구조체

1. 구조체를 정의하려는 헤더(.h) 파일을 연다.
2. 구조체를 정의하고 `USTRUCT` 매크로를 추가한다.
3. 구조체 상단에 `GENERATED_BODY` 매크로를 추가한다.
4. 구조체의 멤버변수를 `UPROPERTY` 로 태그하여 언리얼 엔진의 리플렉션 시스템과 블루프린트에 표시 할 수 있다.

```cpp
USTRUCT(BlueprintType)      // 블루프린트와 호환되는 데이터
struct FMyStruct            // 언리얼 오브젝트가 아니라서 F로 시작
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWirte, Category="Test Variables")
    int32 MyIntegerMemberVariable;

    int32 NativeOnlyMemberVariable; // 이 멤버 변수는 블루프린트 그래프를 통해 액세스 불가

    UPROPERTY()
    UObject* SafeObjectPointer;
    // 구조체에 언리얼 포인터를 선언하게 되면, UPROPERTY를 붙여줘야
    // 자동으로 메모리 관리 가능
}
```

- 데이터 저장/전송에 특화된 가벼운 객체
- 대부분 `GENERATED_BODY` 매크로를 선언해준다.
  - 리플렉션, 직렬화와 같은 유용한 기능 지원
  - `GENERATED_BODY` 를 선언한 구조체는 UScriptStruct 클래스로 구현됨
  - 이 경우 제한적으로 리플렉션을 지원함
    - 속성 `UPROPERTY` 만 선언할 수 있고, 함수 `UFUNCTION` 은 선언 불가
  - 구조체 이름은 <b>F</b> 로 시작함
    - 대부분 힙 메모리 할당(포인터 연산) 없이 스택 내 데이터로 사용됨

## 코드

### Student.h

```cpp
UCLASS()
class UNREALCONTAINER_API UStudent : public UObject // UObject를 상속받은 언리얼 클래스
{
	GENERATED_BODY()
	
};
```

### MyGameInstance.h

```cpp
USTRUCT()
struct FStudentData
{
	GENERATED_BODY()

	FStudentData()
	{
		Name = TEXT("홍길동");
		Order = -1;
	}

	FStudentData(FString InName, int32 InOrder) : Name(InName), Order(InOrder) {}

	UPROPERTY()
	FString Name;		// 리플렉션을 통해 관리하기 위해 UPROPERTY 붙여줌

	UPROPERTY()
	int32 Order;
};

UCLASS()
class UNREALCONTAINER_API UMyGameInstance : public UGameInstance
{
	GENERATED_BODY()
	
public:
	virtual void Init() override;

private:
	TArray<FStudentData> StudentsData;

	UPROPERTY()
	TArray<TObjectPtr<class UStudent>> Students;
	// 언리얼 오브젝트를 TArray로 다룰 땐 반드시! UPROPERTY()!
};
```


### MyGameInstance.cpp

```cpp
#include "MyGameInstance.h"
#include "Algo/Accumulate.h"

FString MakeRandomName()
{
	TCHAR FirstChar[] = TEXT("김이박최");
	TCHAR MiddleChar[] = TEXT("상혜지성");
	TCHAR LastChar[] = TEXT("수은원연");

	TArray<TCHAR> RandArray;
	RandArray.SetNum(3);
	RandArray[0] = FirstChar[FMath::RandRange(0, 3)];
	RandArray[1] = MiddleChar[FMath::RandRange(0, 3)];
	RandArray[2] = LastChar[FMath::RandRange(0, 3)];

	return RandArray.GetData();
}

void UMyGameInstance::Init()
{
	Super::Init();

	const int32 StudentNum = 300;
	for (int32 ix = 1; ix <= StudentNum; ++ix)
	{
		StudentsData.Emplace(FStudentData(MakeRandomName(), ix));
	}

	TArray<FString> AllStudentsNames;

	// StudentsData의 Name들을 AllStudentsName 배열로 옮기기
	Algo::Transform(StudentsData, AllStudentsNames,
		[](const FStudentData& Val)
		{
			return Val.Name;
		});

	UE_LOG(LogTemp, Log, TEXT("모든 학생 이름의 수 : %d"), AllStudentsNames.Num());

	TSet<FString> AllUniqueNames;			// TSet의 경우 중복 허용 안 함.
	Algo::Transform(StudentsData, AllUniqueNames,
		[](const FStudentData& Val)
		{
			return Val.Name;
		});

	UE_LOG(LogTemp, Log, TEXT("중복 없는 학생 이름의 수 : %d"), AllUniqueNames.Num());
}
```



# 2. Map

[STL Map 정리 링크](https://eggmong.github.io/cpp/1-STL-Map/#1-%EB%A7%B5-map)

- key, value 구성의 tuple 데이터의 TSet 구조로 구현됨
- 그래서 해시테이블 형태로 구축되어 있어 빠름
- 동적 배열의 형태로 데이터가 모여 있음
- 빈 데이터가 있을 수 있음
- 데이터를 삭제한다고 재구축이 일어나진 않음
- `TMultiMap` 을 사용하면 중복 데이터를 허용하는 맵 생성


## 코드

### MyGameInstance.h

```cpp
USTRUCT()
struct FStudentData
{
	GENERATED_BODY()

	FStudentData()
	{
		Name = TEXT("홍길동");
		Order = -1;
	}

	FStudentData(FString InName, int32 InOrder) : Name(InName), Order(InOrder) {}

	// TSet과 TMap에서 언리얼 구조체를 사용하기 위해선 함수를 추가해줘야 함
	bool operator==(const FStudentData& InOther) const
	{
		return Order == InOther.Order;
	}

	friend FORCEINLINE uint32 GetTypeHash(const FStudentData& InStudentData)
	{
		return GetTypeHash(InStudentData.Order);
	}

	UPROPERTY()
	FString Name;		// 리플렉션을 통해 관리하기 위해 UPROPERTY 붙여줌

	UPROPERTY()
	int32 Order;
};

/**
 * 
 */
UCLASS()
class UNREALCONTAINER_API UMyGameInstance : public UGameInstance
{
	GENERATED_BODY()
	
public:
	virtual void Init() override;

private:
	TArray<FStudentData> StudentsData;

	UPROPERTY()
	TArray<TObjectPtr<class UStudent>> Students;
	// 언리얼 오브젝트를 TArray로 다룰 땐 반드시! UPROPERTY()!

	TMap<int32, FString> StudentsMap;
};
```

#### GetTypeHash : 언리얼 구조체를 TSet, TMap에서 사용하려면.



### MyGameInstance.cpp

```cpp
Algo::Transform(StudentsData, StudentsMap,
    [](const FStudentData& Val)
    {
        return TPair<int32, FString>(Val.Order, Val.Name);
    });

UE_LOG(LogTemp, Log, TEXT("순번에 따른 학생 맵의 레코드 수 : %d"), StudentsMap.Num());

TMap<FString, int32> StudentsMapByUniqueName;

Algo::Transform(StudentsData, StudentsMapByUniqueName,
    [](const FStudentData& Val)
    {
        return TPair<FString, int32>(Val.Name, Val.Order);
    });

UE_LOG(LogTemp, Log, TEXT("이름에 따른 학생 맵의 레코드 수 : %d"), StudentsMapByUniqueName.Num());

TMultiMap<FString, int32> StudentMapByName;
Algo::Transform(StudentsData, StudentMapByName,
    [](const FStudentData& Val)
    {
        return TPair<FString, int32>(Val.Name, Val.Order);
    });

UE_LOG(LogTemp, Log, TEXT("이름에 따른 학생 멀티맵의 레코드 수 : %d"), StudentMapByName.Num());

const FString TargetName(TEXT("이혜은"));
TArray<int32> AllOrders;
StudentMapByName.MultiFind(TargetName, AllOrders);

UE_LOG(LogTemp, Log, TEXT("이름이 %s인 학생 수 : %d"), *TargetName, AllOrders.Num());

TSet<FStudentData> StudentsSet;
for (int32 ix = 1; ix <= StudentNum; ++ix)
{
    StudentsSet.Emplace(FStudentData(MakeRandomName(), ix));
}
```

