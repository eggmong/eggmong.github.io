---
title:  "언리얼 메모 5++ / 언리얼 프로퍼티 시스템 (리플렉션)"
expert: "리플렉션, Reflection"

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

- 객체 생성 : `NewObject`
- FindPropertyByName
- FindFunctionByName
- GetValue_InContainer
- SetValue_InContainer
- ProcessEvent

```cpp
// MyGameInstance.h
#pragma once

#include "CoreMinimal.h"
#include "Engine/GameInstance.h"
#include "MyGameInstance.generated.h"

UCLASS()
class OBJECTREFLECTION_API UMyGameInstance : public UGameInstance
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

```cpp
#include "MyGameInstance.h"         // 메인 인스턴스의 헤더가 항상 위에 있게..
#include "Student.h"
#include "Teacher.h"

UMyGameInstance::UMyGameInstance()
{
	SchoolName = TEXT("Default 대학교");
}

void UMyGameInstance::Init()
{
	UStudent* Student = NewObject<UStudent>();
	UTeacher* Teacher = NewObject<UTeacher>();

	// 만들어둔 프로퍼티로 이름 가져오기
	Student->SetName(TEXT("학생1"));
	UE_LOG(LogTemp, Log, TEXT("새로운 학생 이름 %s"), *Student->GetName());

	FString NewTeacherName(TEXT("이땡땡"));
	
	// 이렇게도 Name을 가져올 수 있다
	FString CurrentTeacherName;
	FProperty* NameProp = UTeacher::StaticClass()->FindPropertyByName(TEXT("Name"));
	// Name 프로퍼티 긁어오는 것

	if (NameProp)
	{
		// 이렇게도 Name을 가져올 수 있다
		NameProp->GetValue_InContainer(Teacher, &CurrentTeacherName);
		UE_LOG(LogTemp, Log, TEXT("현재 선생님 이름 %s"), *CurrentTeacherName);

		// 이름 바꿔보기
		NameProp->SetValue_InContainer(Teacher, &NewTeacherName);
		UE_LOG(LogTemp, Log, TEXT("현재 선생님 이름 %s"), *Teacher->GetName());
	}

	UE_LOG(LogTemp, Log, TEXT("======================="));

	Student->DoLesson();
	UFunction* DoLessonFunc = Teacher->GetClass()->FindFunctionByName("DoLesson");
	if (DoLessonFunc)
	{
		Teacher->ProcessEvent(DoLessonFunc, nullptr);
	}
}
```

***

```cpp
// Person.h
#pragma once

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "Person.generated.h"

UCLASS()
class OBJECTREFLECTION_API UPerson : public UObject
{
	GENERATED_BODY()
	
public:
	UPerson();

	UFUNCTION()
	virtual void DoLesson();

	const FString& GetName() const;
	void SetName(const FString& InName);

protected:
	UPROPERTY()
	FString Name;

	UPROPERTY()
	int32 Year;
private:
};
```

```cpp
// Person.cpp
#include "Person.h"

UPerson::UPerson()
{
	Name = TEXT("홍길동");
	Year = 1;
}

void UPerson::DoLesson()
{
	UE_LOG(LogTemp, Log, TEXT("%s 님이 수업에 참여합니다."), *Name);
}

const FString& UPerson::GetName() const
{
	return Name;
}

void UPerson::SetName(const FString& InName)
{
	Name = InName;
}
```

```cpp
// Student.h
#pragma once

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "Person.h"                 
#include "Student.generated.h"  // .generated 헤더 포함하는 코드는 항상 맨 마지막 줄에

UCLASS()
class OBJECTREFLECTION_API UStudent : public UPerson
{
	GENERATED_BODY()
public:
	UStudent();

	virtual void DoLesson() override;
private:
	UPROPERTY()
	int32 Id;
};
```

```cpp
// Student.cpp
#include "Student.h"

UStudent::UStudent()
{
	Name = TEXT("이학생");
	Year = 1;
	Id = 1;
}

void UStudent::DoLesson()
{
	Super::DoLesson();

	UE_LOG(LogTemp, Log, TEXT("%d학년 %d번 %s님이 수업을 듣습니다"), Year, Id, *Name);
}
```

```cpp
// Teacher.h
#pragma once

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "Person.h"
#include "Teacher.generated.h"

UCLASS()
class OBJECTREFLECTION_API UTeacher : public UPerson
{
	GENERATED_BODY()
public:
	UTeacher();

	virtual void DoLesson() override;
private:
	int32 Id;
};
```

```cpp
// Teacher.cpp
UTeacher::UTeacher()
{
	Name = TEXT("이선생");
	Year = 3;
	Id = 1;
}

void UTeacher::DoLesson()
{
	Super::DoLesson();

	UE_LOG(LogTemp, Log, TEXT("%d연차 선생님 %s님이 수업을 강의합니다."), Year, *Name);
}
```