---
title:  "[정리] 언리얼 메모 6 / 인터페이스, 형변환"
expert: "인터페이스, Interface"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: true
toc_sticky: true
 
date: 2024-01-04
last_modified_at: 2024-01-04
---

<br>

언리얼 엔진에서 게임 컨텐츠를 구성하는 오브젝트의 설계 예시
- 월드에 배치되는 모든 오브젝트 (안 움직이는 오브젝트 포함. Actor)
- 움직이는 오브젝트 (Pawn)
- 길찾기 시스템을 <b>반드시</b> 사용하면서 움직이는 오브젝트
  - INavAgentInterface 인터페이스를 구현한 Pawn


# 언리얼 C++ 인터페이스 특징

- 인터페이스를 생성하면 두 개의 클래스가 생성됨
  - `U`로 시작하는 타입 클래스
  - `I`로 시작하는 인터페이스 클래스
- <b>객체를 설계할 때</b> `I` 인터페이스 클래스를 사용
  - `U`타입 클래스 정보는 런타임에서 인터페이스 구현 여부를 파악하는 용도로 사용됨
  - 실제로 `U`타입 클래스에서 작업할 일은 <b>없음</b>
  - 인터페이스에 관련된 구성 및 구현은 `I`인터페이스 클래스에서 진행
- C++ 인터페이스의 특징
  - 추상 타입으로만 선언할 수 있는 Java, C#과 달리 언리얼은 <b>인터페이스에도 구현 가능</b>

- 클래스가 반드시 구현해야 하는 기능을 지정하는데에 사용해보자



# MyGameInstance.h

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Engine/GameInstance.h"
#include "MyGameInstance.generated.h"

UCLASS()
class UNREALINTERFACE_API UMyGameInstance : public UGameInstance
{
	GENERATED_BODY()
	
public:
	UMyGameInstance();
	virtual void Init() override;

private:
	UPROPERTY()
	FString SchoolName;
};
```

# MyGameInstance.cpp

```cpp
#include "MyGameInstance.h"
#include "Student.h"
#include "Teacher.h"
#include "Staff.h"

UMyGameInstance::UMyGameInstance()
{
	SchoolName = TEXT("Default 대학교");
}

void UMyGameInstance::Init()
{
	Super::Init();

	TArray<UPerson*> Persons = {
		NewObject<UStudent>(), NewObject<UTeacher>(), NewObject<UStaff>()
	};

	for (const auto Person : Persons)
	{
		UE_LOG(LogTemp, Log, TEXT("구성원 이름 : %s"), *Person->GetName());
	}

	for (const auto Person : Persons)
	{
		ILessonInterface* LessonInterface = Cast<ILessonInterface>(Person);

		if (LessonInterface)	// 형변환 성공, 인터페이스 갖고있다는 소리
		{
			UE_LOG(LogTemp, Log, TEXT("%s님은 수업에 참여할 수 있습니다."), *Person->GetName());
			LessonInterface->DoLesson();
		}
		else
		{
			UE_LOG(LogTemp, Log, TEXT("%s님은 수업에 참여할 수 없습니다."), *Person->GetName());
		}
	}
}
```

## Cast<>()

> 💡 Cast 함수로 인터페이스 갖고있으면 형변환해서 가져올 수 있다.

***

# LessonIntereface.h

```cpp
#pragma once

#include "CoreMinimal.h"
#include "UObject/Interface.h"
#include "LessonInterface.generated.h"

// This class does not need to be modified.
UINTERFACE(MinimalAPI)
class ULessonInterface : public UInterface
{
	GENERATED_BODY()
};


class UNREALINTERFACE_API ILessonInterface
{
	GENERATED_BODY()

public:
	virtual void DoLesson()
	{
		// 언리얼 인터페이스는 구현도 가능함! 비워두지 않아도 됨

		UE_LOG(LogTemp, Log, TEXT("수업에 입장합니다."));
	}
};
```

## 언리얼 인터페이스는 내부 구현도 가능함

cpp엔 딱히 구현 안했음. 스킵  

***

# Person.h

```cpp
#pragma once

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "Person.generated.h"

UCLASS()
class UNREALINTERFACE_API UPerson : public UObject
{
	GENERATED_BODY()
	
public:
	UPerson();

	// FORCEINLINE : 인라인 지정
	FORCEINLINE FString& GetName() { return Name; }
	FORCEINLINE void SetName(const FString& InName) { Name = InName; }

protected:
	UPROPERTY()
	FString Name;
};

```

## FORCEINLINE : 인라인 지정

# Person.cpp

```cpp
#include "Person.h"

UPerson::UPerson()
{
	Name = TEXT("홍길동");
}
```

***

# Student.h

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Person.h"
#include "LessonInterface.h"
#include "Student.generated.h"

UCLASS()
class UNREALINTERFACE_API UStudent : public UPerson, public ILessonInterface
{
	GENERATED_BODY()
	
public:
	UStudent();

	virtual void DoLesson() override;
};
```

# Student.cpp

```cpp
UStudent::UStudent()
{
	Name = TEXT("이학생");
}

void UStudent::DoLesson()
{
	// 인터페이스에서 구현한 걸 출력하고 싶다면
	// Super::DoLesson(); 떠올렸겠지만 아님...
	// 학생의 부모는 Person임. LessonInterface가 아니라.
	// 이렇게 작성
	ILessonInterface::DoLesson();

	UE_LOG(LogTemp, Log, TEXT("%s님은 공부합니다."), *Name);
}
```

## 인터페이스의 함수 호출 방법

> 💡 인터페이스에서 구현한 걸 출력하고 싶다면  
> Super::DoLesson(); 떠올렸겠지만 아님...  
> 학생의 부모는 Person임. LessonInterface가 아니라.  
> `ILessonInterface::DoLesson();`

***

# Teacher.h

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Person.h"
#include "LessonInterface.h"
#include "Teacher.generated.h"

UCLASS()
class UNREALINTERFACE_API UTeacher : public UPerson, public ILessonInterface
{
	GENERATED_BODY()
	
public:
	UTeacher();

	virtual void DoLesson() override;
};
```

# Teacher.cpp

```cpp
#include "Teacher.h"

UTeacher::UTeacher()
{
	Name = TEXT("이선생");
}

void UTeacher::DoLesson()
{
	ILessonInterface::DoLesson();

	UE_LOG(LogTemp, Log, TEXT("%s님은 가르칩니다."), *Name);
}
```

***

# Staff.h

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Person.h"
#include "Staff.generated.h"

UCLASS()
class UNREALINTERFACE_API UStaff : public UPerson
{
	GENERATED_BODY()
	
public:
	UStaff();
};
```

# Staff.cpp

```cpp
#include "Staff.h"

UStaff::UStaff()
{
	Name = TEXT("이직원");
}
```