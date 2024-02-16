---
title:  "[정리] 언리얼 메모 5 / 언리얼 프로퍼티 시스템 (리플렉션)"
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


# 언리얼 프로퍼티 시스템 (리플렉션)

[공식문서 링크](https://www.unrealengine.com/ko/blog/unreal-property-system-reflection)


> 프로그램이 실행시간에 자기 자신을 조사하는 기능


```cpp
// 예시
// Base class for mobile units (soldiers)
#include "StrategyTypes.h"
#include "StrategyChar.generated.h"

UCLASS(Abstract)
class AStrategyChar : public ACharacter, public IStrategyTeamInterface
{
GENERATED_BODY()

/** How many resources this pawn is worth when it dies. */
UPROPERTY(EditAnywhere, Category=Pawn)
int32 ResourcesToGather;

/** set attachment for weapon slot */
UFUNCTION(BlueprintCallable, Category=Attachment)
void SetWeaponAttachment(class UStrategyAttachment* Weapon);

UFUNCTION(BlueprintCallable, Category=Attachment)
bool IsWeaponAttached();

protected:
/** melee anim */
UPROPERTY(EditDefaultsOnly, Category=Pawn) // <-------- 이거!!!
```

> `EditDefaultsOnly, Category=Pawn` 같은 특별한 지시자 : 추가적인 메타 데이터  
> 에디터와 직접 연동하여 게임 제작할 때 사용됨  


> 리플렉션된 프로퍼티가 아닌 것들은 언리얼 엔진 시스템에서 관리되지 않음 -> 직접 처리해야 함  
> `UPROPERTY` 라고 위에 선언해줘야 언리얼이 메모리 관리를 해줌 (가비지 컬렉션)  

## 언리얼 오브젝트의 구성

- 언리얼 오브젝트에는 특별한 프로퍼티와 함수를 지정할 수 있음
  - 관리되는 클래스 멤버 변수 : UPROPERTY
  - 관리되는 클래스 멤버 함수 : UFUNCTION

- 모든 언리얼 오브젝트는 클래스 정보와 함께 함 (UClass)
  - `UClass` 를 사용해 자신이 가진 프로퍼티와 함수 정보를 컴파일 타임과 런타임에서 조회할 수 있음

- 이렇게 언리얼 오브젝트는 다양한 기능을 제공하기 때문에,  
  일반적인 C++ 객체를 생성할 때 사용되는 `new() 키워드` 가 아니라 별도의 `NewObject()` 라는  
  API를 제공해서 생성해야 함

![UO]({{ site.imageurl }}Images/UnrealObject.png)  



## 언리얼 오브젝트의 클래스 기본 오브젝트

- 언리얼 클래스 정보에는 클래스 기본 오브젝트 (`CDO`, Class Default Object) 가 함께 포함되어 있음
- `CDO` 는 언리얼 객체가 가진 기본 값을 보관하는 템플릿 객체
- 한 클래스로부터 다수의 물체를 생성해 게임 콘텐츠에 배치할 때,  
  일관성 있게 기본값을 조정하는 데에 유용하게 사용됨
- `CDO` 는 클래스 정보로부터 `GetDefaultObject` 함수를 통해 얻을 수 있음
- UClass 및 `CDO` 는 엔진 초기화 과정에서 생성되므로 콘텐츠 제작에서 안심하고 사용 가능

![CDO]({{ site.imageurl }}Images/
UnrealClassDefaultObject.png)  



## 언리얼 오브젝트의 처리

[언리얼 오브젝트의 처리 공식 문서](https://docs.unrealengine.com/5.3/ko/unreal-object-handling-in-unreal-engine/)

> 클래스, 프로퍼티, 함수에 적합한 매크로로 마킹해 주면 UClass, UProperty, UFunction 으로 변합니다.  
> 그러면 언리얼 엔진이 접근할 수 있어, 다수의 내부적인 처리 기능을 구현할 수 있습니다.  


```cpp
#pragma once

#include "CoreMinimal.h"
#include "Engine/GameInstance.h"
#include "MyGameInstance.generated.h"

/**
 * 
 */
UCLASS()
class OBJECTREFLECTION_API UMyGameInstance : public UGameInstance
{
	GENERATED_BODY()
	
public:
	UMyGameInstance();
	virtual void Init() override;

private:
	UPROPERTY()     // SchoolName을 CDO로 지정하기 위해 프로퍼티 붙임요
	FString SchoolName;
};
```

```cpp
#include "MyGameInstance.h"

UMyGameInstance::UMyGameInstance()
{
	SchoolName = TEXT("Default 대학교");
}

void UMyGameInstance::Init()
{
	Super::Init();

	UE_LOG(LogTemp, Log, TEXT("================"));
	
	UClass* ClassRunTime = GetClass();  // UMyGameInstance에 대한 클래스 정보 얻어오기
                                        // (런타임으로)
	UClass* ClassCompile = UMyGameInstance::StaticClass();	// UMyGameInstance에 대한 클래스 정보 얻어오기
                                                            // (컴파일때)
	// 위 두 코드는 모두 동일한 객체를 가리키고 있다

	check(ClassRunTime == ClassCompile);    // 언리얼 Assertion
                                            // 조건에 부합하지 않으면 에디터가 뻗으면서
                                            // 크래시 리포트 출력
                                            // != 로 바꾸면 리포트가 발생함.
                                            // 같은건데 아니라고 하니까;

	ensure(ClassRunTime != ClassCompile);   // check 를 사용하면 에디터가 꺼져버리는데,
                                            // 그게 번거롭다면 이걸 사용
                                            // 에디터가 뻗지 않고, 로그 출력함

	ensureMsgf(ClassRunTime != ClassCompile, TEXT("일부러 에러를 발생시킨 코드"));

	UE_LOG(LogTemp, Log, TEXT("학교를 담당하는 클래스 이름 : %s"), *ClassRunTime->GetName());
    // 출력: 학교를 담당하는 클래스 이름 : MyGameInstance

	SchoolName = TEXT("1번 대학교");

	UE_LOG(LogTemp, Log, TEXT("학교 이름 : %s"), *SchoolName);  // 1번 대학교 출력

	UE_LOG(LogTemp, Log, TEXT("Default 학교 이름 : %s"),
    *GetClass()->GetDefaultObject<UMyGameInstance>()->SchoolName);
	// 위의 코드는 학교 이름을 출력하지 않는다...
	// CDO의 경우 에디터가 활성화 되기 이전에 초기화 되기 때문에,
	// 생성자에서 초기화를 해도 에디터에서 인지를 못 할 수 있음

	// CDO 를 고치는 생성자 코드를 변경할 땐
	// 헤더 파일을 변경할 때 처럼
	// 에디터를 끄고 컴파일을 진행 후 에디터를 켜야 한다
	// 그러면 다시 잘 출력 됨

	UE_LOG(LogTemp, Log, TEXT("================"));
}
```


## 정리

- 언리얼 오브젝트에는 항상 클래스 정보를 담은 UClass 객체가 매칭되어 있다
- UClass로부터 언리얼 오브젝트의 정보를 파악할 수 있음
- UClass에는 클래스 기본 오브젝트(CDO)가 연결되어 있어 이를 활용할 수 있음
- 클래스 정보와 CDO는 엔진 초기화 과정에서 생성됨
- <b>헤더 정보를 변경하거나, 생성자 정보를 변경하면</b> 에디터를 끄고 컴파일해야 함