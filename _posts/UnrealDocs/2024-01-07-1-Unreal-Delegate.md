---
title:  "언리얼 설계 - 델리게이트"
expert: "델리게이트, Delegate"

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

언리얼 델리게이트를 사용하여 클래스 간의 느슨한 결합을 구현 가능  

- 강한 결합
  - 클래스들이 서로 의존성을 가지는 경우를 뜻함


  ```cpp
  class Card
  {
    public:
        Card(int InId) : Id(InId) {}
        int Id = 0;
  };

  class Person
  {
    public:
        Person(Card InCard) : IdCard(InCard) {}
    protected:
        Card IdCard;        //  멤버변수로 직접 Card 클래스 사용
  };
  ```
  
  - 위 예시 코드에서 Card 클래스가 없는 경우 Person은 만들어질 수 없다.

- 느슨한 결합
  - 실물에 의존하지 말고 추상적 설계에 의존하라 (DIP 원칙)
  - ICheck를 상속 받은 새로운 카드 인터페이스를 선언하여 해결하자

  ```cpp
  class ICheck
  {
    public:
        virtual bool check() = 0;
  };

  class Card : public ICheck
  {
    public:
        Card(int InId) : Id(InId) {}
        virtual bool check() { return true; }
    private:
        int Id = 0;
  }

  class Person
  {
    public:
        Person(ICheck* InCheck) : Check(InCheck) {}
    protected:
        ICheck* Check;
  };
  ```


> 하지만 매번 인터페이스를 만드는 것이 번거로울 수 있음.  
> 그렇다면 어떤 행동에 대해서, 즉 함수를 오브젝트처럼 관리한다면?  

```cpp
public class Card
{
    public int Id;
    public bool CardCheck() {return true;}
}

public delegate bool CheckDelegate();
public class Person
{
    Person(CheckDelegate InCheckDelegate)
    {
        this.Check = InCheckDelegate;
    }

    public CheckDelegate Check;
}
```


# 델리게이트 (Delegate)

[언리얼 델리게이트 공식 문서 링크](https://docs.unrealengine.com/4.27/ko/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/)

데리게이트 오브젝트는 복사해도 완벽히 안전하다.  

- 델리게이트 유형
  - DECLARE_DELEGATE
    - 일대일 형태로 C++만 지원한다면
  - DECLARE_MULTICAST
    - 일대다 형태로 c++만 지원한다면
  - DECLARE_DYNAMIC
    - 일대일 형태로 블루프린트를 지원한다면
  - DECLARE_DYNAMIC_MULTICAST
    - 일대다 형태로 블루프린트를 지원한다면...

- 함수 정보
  - 인자가 없고 반환값도 없으면 공란 (DECLARE_DELEGATE)
  - 인자가 하나고 반환값이 없으면 OneParam으로 지정 (DECLARE_DELEGATE_OneParam)
  - 인자가 세 개고 반환값이 있으면 RetVal_ThreeParams
  - 최대 9개까지 지원



![Delegate](https://github.com/eggmong/eggmongImages/raw/main/Images/
UnrealDeleagte.png)  


### CourseInfo.h

```cpp
DECLARE_MULTICAST_DELEGATE_TwoParams(FCourseInfoOnChangedSignature, const FString&, const FString&);

UCLASS()
class UNREALDELEGATE_API UCourseInfo : public UObject
{
	GENERATED_BODY()
	
public:
	UCourseInfo();
	FCourseInfoOnChangedSignature OnChanged;

	void ChangeCourseInfo(const FString& InSchoolName, const FString& InNewContents);

private:
	FString Contents;
};
```

### CourseInfo.cpp

```cpp
#include "CourseInfo.h"

UCourseInfo::UCourseInfo()
{
	Contents = TEXT("기존 학사 정보");
}

void UCourseInfo::ChangeCourseInfo(const FString& InSchoolName, const FString& InNewContents)
{
	Contents = InNewContents;

	UE_LOG(LogTemp, Log, TEXT("[CourseInfo] 학사 정보가 변경되어 알림을 발송합니다."));
	OnChanged.Broadcast(InSchoolName, Contents);
}
```

### Student.h

```cpp
UCLASS()
class UNREALDELEGATE_API UStudent : public UPerson, public ILessonInterface
{
	GENERATED_BODY()
	
public:
	UStudent();

	virtual void DoLesson() override;

	void GetNotification(const FString& School, const FString& NewCourseInfo);
};
```

### Student.cpp

```cpp
#include "Student.h"
#include "Card.h"

UStudent::UStudent()
{
	Name = TEXT("이학생");
}

void UStudent::GetNotification(const FString& School, const FString& NewCourseInfo)
{
	UE_LOG(LogTemp, Log, TEXT("[Student] %s님이 %s로부터 받은 메시지 : %s"), *Name, *School, *NewCourseInfo);
}
```

### MyGameInstance.h

```cpp
UCLASS()
class UNREALDELEGATE_API UMyGameInstance : public UGameInstance
{
	GENERATED_BODY()
	
public:
	UMyGameInstance();
	virtual void Init() override;

private:
	UPROPERTY()			// 언리얼 오브젝트
	TObjectPtr<class UCourseInfo> CourseInfo;	// 포인터로 관리해줘야 하기 때문에
												// 전방선언! class ! 그리고
												// 언리얼 오브젝트의 포인터를 멤버 변수로 지정할 땐 TObjectPtr

	UPROPERTY()
	FString SchoolName;
};
```

### MyGameInstance.cpp

```cpp
#include "CourseInfo.h"

UMyGameInstance::UMyGameInstance()
{
	SchoolName = TEXT("학교");
}

void UMyGameInstance::Init()
{
	Super::Init();

	CourseInfo = NewObject<UCourseInfo>(this);

	UE_LOG(LogTemp, Log, TEXT("============================"));

	UStudent* Student1 = NewObject<UStudent>();
	Student1->SetName(TEXT("학생1"));
	UStudent* Student2 = NewObject<UStudent>();
	Student2->SetName(TEXT("학생2"));
	UStudent* Student3 = NewObject<UStudent>();
	Student3->SetName(TEXT("학생3"));

	CourseInfo->OnChanged.AddUObject(Student1, &UStudent::GetNotification);
	CourseInfo->OnChanged.AddUObject(Student2, &UStudent::GetNotification);
	CourseInfo->OnChanged.AddUObject(Student3, &UStudent::GetNotification);

	CourseInfo->ChangeCourseInfo(SchoolName, TEXT("변경된 학사 정보"));

	UE_LOG(LogTemp, Log, TEXT("============================"));
}
```