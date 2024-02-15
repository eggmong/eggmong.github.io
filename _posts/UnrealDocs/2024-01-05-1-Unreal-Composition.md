---
title:  "언리얼 설계 - 컴포지션"
expert: "컴포지션, Composition"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: true
toc_sticky: true
 
date: 2024-01-05
last_modified_at: 2024-01-05
---

<br>

> 컴포지션은 객체 지향 설계에서 Has-A 관계를 구현하는 설계 방법

# 컴포지션 구현 방법

- 한 언리얼 오브젝트에는 항상 클래스 기본 오브젝트 CDO가 있다.
- 언리얼 오브젝트간의 컴포지션은 어떻게 구현할 것인가?
  - 1. CDO에 미리 언리얼 오브젝트를 생성해 조합한다. (필수적 조합)
  - 2. CDO에 빈 포인터만 넣고 런타임에서 언리얼 오브젝트를 생성해 조합한다. (선택적 조합)
- 언리얼 오브젝트를 생성할 때 컴포지션 정보를 구축할 수 있다.
  - 내가 소유한 언리얼 오브젝트를 Subobject라고 한다.
  - 나를 소유한 언리얼 오브젝트를 Outer라고 한다.

![Composition]({{ site.imageurl }}Images/
UnrealComposition.png)  


## 코드

### Card.h
#### UMETA

```cpp
UENUM()
enum class ECardType : uint8
{
	Student =  1 UMETA(DisplayName = "For Student"),
	Teacher UMETA(DisplayName = "For Teacher"),
	Staff UMETA(DisplayName = "For Staff"),
	Invalid
};

UCLASS()
class UNREALCOMPOSITION_API UCard : public UObject
{
	GENERATED_BODY()
	
public:
	UCard();

	ECardType GetCardType() const { return CardType; }
	void SetCardType(ECardType InCardType) { CardType = InCardType; }

private:
	UPROPERTY()
	ECardType CardType;

	UPROPERTY()
	uint32 Id;
};
```

### Card.cpp

```cpp
#include "Card.h"

UCard::UCard()
{
	CardType = ECardType::Invalid;
	Id = 0;
}
```


### LessonInterface.h

```cpp
// This class does not need to be modified.
UINTERFACE(MinimalAPI)
class ULessonInterface : public UInterface
{
	GENERATED_BODY()
};

class UNREALCOMPOSITION_API ILessonInterface
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

LessonInterface.cpp 는 수정 안 했음.  

[인터페이스 U 붙은건 안건드린다! 링크](https://eggmong.github.io/unrealdocs/3-Unreal-Interface/#%EC%96%B8%EB%A6%AC%EC%96%BC-c-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4-%ED%8A%B9%EC%A7%95)



### Person.h

```cpp
UCLASS()
class UNREALCOMPOSITION_API UPerson : public UObject
{
	GENERATED_BODY()
	
public:
	UPerson();

	// FORCEINLINE : 인라인 지정
	FORCEINLINE const FString& GetName() const { return Name; }
	FORCEINLINE void SetName(const FString& InName) { Name = InName; }

	FORCEINLINE class UCard* GetCard() const { return Card; }
	FORCEINLINE void SetCard(class UCard* InCard) { Card = InCard; }

protected:
	UPROPERTY()
	FString Name;

	// 전방선언. 헤더를 포함하지 않고도 포인터 선언 가능.
	// 헤더를 포함하지 않아 구체적인 구현부는 몰라도
	// 포인터 크기는 알 수 있다.
	// 보통 오브젝트는 포인터로 관리하기 때문에...
	// 이렇게 전방선언 하여 의존성을 낮출 수 있다.
	/*UPROPERTY()
	class UCard* Card;*/

	// 그런데 언리얼5 버전부턴 원시 포인터를 다르게 쓰라고 에픽에서 제안함.
	UPROPERTY()
	TObjectPtr<class UCard> Card;

};
```

#### 전방선언 및 TObjectPtr

`TObjectPtr<class UCard> Card;`

### Person.cpp

```cpp
#include "Person.h"
#include "Card.h"

UPerson::UPerson()
{
	Name = TEXT("홍길동");
	Card = CreateDefaultSubobject<UCard>(TEXT("NAME_Card"));	// FName
}
```


### Teacher.h

```cpp
#include "Person.h"
#include "LessonInterface.h"

UCLASS()
class UNREALCOMPOSITION_API UTeacher : public UPerson, public ILessonInterface
{
	GENERATED_BODY()
	
public:
	UTeacher();

	virtual void DoLesson() override;
};
```

### Teacher.cpp

```cpp
#include "Teacher.h"
#include "Card.h"

UTeacher::UTeacher()
{
	Name = TEXT("이선생");
	Card->SetCardType(ECardType::Teacher);
}

void UTeacher::DoLesson()
{
	ILessonInterface::DoLesson();

	UE_LOG(LogTemp, Log, TEXT("%s님은 가르칩니다."), *Name);
}
```


### Student.h

```cpp
#include "Person.h"
#include "LessonInterface.h"

UCLASS()
class UNREALCOMPOSITION_API UStudent : public UPerson, public ILessonInterface
{
	GENERATED_BODY()
	
public:
	UStudent();

	virtual void DoLesson() override;
};
```

```cpp
#include "Student.h"
#include "Card.h"

UStudent::UStudent()
{
	Name = TEXT("이학생");
	Card->SetCardType(ECardType::Student);
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

#### 인터페이스에서 작성된 함수를 호출하는 법

`ILessonInterface::DoLesson();`


### Staff.h

```cpp
#include "Person.h"
UCLASS()
class UNREALCOMPOSITION_API UStaff : public UPerson
{
	GENERATED_BODY()
	
public:
	UStaff();
};
```


### Staff.cpp

```cpp
#include "Staff.h"
#include "Card.h"

UStaff::UStaff()
{
	Name = TEXT("이직원");
	Card->SetCardType(ECardType::Staff);
}
```



### 언리얼5에서 오브젝트 포인터 가이드

[언리얼5 마이그레이션 문서](https://docs.unrealengine.com/5.0/ko/unreal-engine-5-migration-guide/#c++%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8%ED%8F%AC%EC%9D%B8%ED%84%B0%ED%94%84%EB%A1%9C%ED%8D%BC%ED%8B%B0)

TObjectPtr 로 포인터를 감싸라고 한다.

