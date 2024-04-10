---
title:  "[정리][Fountain(분수대)] GAS / GameplayAbility 사용 예제"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, GAS, GameAbilitySystem]

toc: true
toc_sticky: true
 
date: 2024-02-27
last_modified_at: 2024-02-27
imagefolder: "UnrealDocs/GameAbilitySystem/"
---

<br>

# GameAbility 코드 예제

C++ 로 어빌리티 작성 및 어빌리티를 블루프린트로도 만들어보기

## ABGameplayTag.h

해당 헤더만 인클루드하면 Gameplay Tag에 관련된 것들을  
자동으로 모두 인클루드 하는 형태가 되도록 헤더 구성  

```cpp
#include "GameplayTagContainer.h"

#define ABTAG_ACTOR_ROTATE FGameplayTag::RequestGameplayTag(FName("Actor.Action.Rotate"))
#define ABTAG_ACTOR_ISROTATING FGameplayTag::RequestGameplayTag(FName("Actor.State.IsRotating"))
```

FGameplayTag::RequestGameplayTag 를 통해 태그가 있는지 확인하고 가져올 수 있다.  
define으로 만들어서 하나의 변수처럼 사용하도록 한다.  





## ABGA_Rotate

개체를 회전시키기 위한 어빌리티

### ABGA_Rotate.h

```cpp
UCLASS()
class ARENABATTLEGAS_API UABGA_Rotate : public UGameplayAbility
{
	GENERATED_BODY()
	
public:
	// Gameplay Tag 등록하기 위해 생성자 생성
	UABGA_Rotate();
	
public:
	// UGameplayAbility 들어가보면 선언되어 있는 함수 중 ActivateAbility, CancelAbility 재정의 할거임.

	virtual void ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData) override;

	virtual void CancelAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, bool bReplicateCancelAbility) override;
};
```

### ABGA_Rotate.cpp

```cpp
#include "GA/ABGA_Rotate.h"
#include "GameFramework/RotatingMovementComponent.h"
#include "Tag/ABGameplayTag.h"          // 위에서 따로 작성했던 헤더

UABGA_Rotate::UABGA_Rotate()
{
	AbilityTags.AddTag(ABTAG_ACTOR_ROTATE);		// 이 어빌리티를 대표하는 태그 추가
	ActivationOwnedTags.AddTag(ABTAG_ACTOR_ISROTATING);				// ActivationOwnedTags : 활성화될 때 발동되는 태그.
																	// 활성화 될 때의 상태 태그 추가
}

/// <param name="Handle">현재 발동한 어빌리티를 우리가 직접 처리할 수 있는 핸들 정보가 들어옴</param>
/// <param name="ActorInfo">AABGASFountain 에서 InitAbilityActorInfo 해줄 때 넣었던 오너 인포, 아바타 인포를 담은 구조체</param>
/// <param name="ActivationInfo">어떻게 발동했는지에 대한 정보를 담은 구조체</param>
/// <param name="TriggerEventData">외부에서 발동했다면 어떻게 발동시켰는지에 대한 정보를 담은 구조체</param>
void UABGA_Rotate::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
	Super::ActivateAbility(Handle, ActorInfo, ActivationInfo, TriggerEventData);

	AActor* AvatarActor = ActorInfo->AvatarActor.Get();
	if (AvatarActor)
	{
		URotatingMovementComponent* RotatingMovement = Cast<URotatingMovementComponent>(AvatarActor->GetComponentByClass(URotatingMovementComponent::StaticClass()));

		if (RotatingMovement)
		{
			RotatingMovement->Activate(true);
		}
	}
}

void UABGA_Rotate::CancelAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, bool bReplicateCancelAbility)
{
	Super::CancelAbility(Handle, ActorInfo, ActivationInfo, bReplicateCancelAbility);

	AActor* AvatarActor = ActorInfo->AvatarActor.Get();
	if (AvatarActor)
	{
		URotatingMovementComponent* RotatingMovement = Cast<URotatingMovementComponent>(AvatarActor->GetComponentByClass(URotatingMovementComponent::StaticClass()));

		if (RotatingMovement)
		{
			RotatingMovement->Deactivate();
		}
	}
}

```





## ABGASFountain

ABGA_Rotate GA를 사용할 개체 구현  

### ABGASFountain.h

```cpp
#include "AbilitySystemInterface.h"
// 게임 어빌리티 시스템, GAS 를 추가하기 위해선
// 이것을 소유하고 있는 액터는 어빌리티 시스템 인터페이스(AbilitySystemInterface) 라고 하는
// 인터페이스를 구현해주는 것이 좋다. (일반적인 구현 방법)

UCLASS()
class ARENABATTLEGAS_API AABGASFountain : public AABFountain, public IAbilitySystemInterface
{
	GENERATED_BODY()
	
public:
	AABGASFountain();

	// IAbilitySystemInterface 이 인터페이스는
	// 어빌리티 시스템 컴포넌트(ASC)를 반환해주는 이 함수를 구현해줘야 한다.
	virtual class UAbilitySystemComponent* GetAbilitySystemComponent() const override;

protected:
	virtual void PostInitializeComponents() override;
	virtual void BeginPlay() override;

	// 주기적으로 반복할 타이머 함수 선언
	virtual void TimerAction();

protected:
	// 개체를 회전시키기 위해 URotatingMovementComponent 선언
    // URotatingMovementComponent : 특정 회전 속도로 구성요소의 연속 회전을 수행
	UPROPERTY(VisibleAnywhere, Category=Movement)
	TObjectPtr<class URotatingMovementComponent> RotatingMovement;

	// 타이머를 반복할 주기 변수
	// 에디터에서 값 고칠 수 있도록 EditAnywhere 로 설정
	UPROPERTY(EditAnywhere, Category=Timer)
	float ActionPeriod;

	UPROPERTY(EditAnywhere, Category=GAS)
	TObjectPtr<class UAbilitySystemComponent> ASC;

	// 시작할 때 발동시킬 어빌리티에 대한 정보를 어레이로 구성
	// 모든 GA들은 UGameplayAbility 를 상속받기 때문에 TSubclassOf로 어레이 생성
	UPROPERTY(EditAnywhere, Category=GAS)
	TArray<TSubclassOf<class UGameplayAbility>> StartAbilities;

	// 언리얼 엔진에서 제공하는 타이머 함수 핸들
	FTimerHandle ActionTimerHandle;
};

```


### ABGASFountain.cpp

```cpp
#include "GameFramework/RotatingMovementComponent.h"
#include "AbilitySystemComponent.h"
#include "GameplayAbilitySpec.h"
#include "Tag/ABGameplayTag.h"
#include "Abilities/GameplayAbility.h"

AABGASFountain::AABGASFountain()
{
	ASC = CreateDefaultSubobject<UAbilitySystemComponent>(TEXT("ASC"));

	RotatingMovement = CreateDefaultSubobject<URotatingMovementComponent>(TEXT("RotateMovement"));
	ActionPeriod = 3.0f;
}

UAbilitySystemComponent* AABGASFountain::GetAbilitySystemComponent() const
{
	return ASC;
}

void AABGASFountain::PostInitializeComponents()
{
	Super::PostInitializeComponents();

	// 컴포넌트가 초기화된 직후엔 자동실행 되지 않도록 끄기
	RotatingMovement->bAutoActivate = false;
	RotatingMovement->Deactivate();

	// 액터가 초기화될 때 이 어빌리티 시스템 컴포넌트도 함께 초기화해줘야 함. 그래서 해당 함수 호출.
	// 오너 정보랑 아타바 정보에 둘 다 this 넣어줬음.
	// 오너 정보 : 실제 ASC를 구동하고 데이터를 관리하는, 실제 작업이 일어나고 있는 액터 정보
	// 아바타 정보 : 실제로 데이터를 처리하진 않지만 비주얼만 수행해주는 액터를 지정
	ASC->InitAbilityActorInfo(this, this);

	for (const auto& StartAbility : StartAbilities)
	{
		FGameplayAbilitySpec StartSpec(StartAbility);
		ASC->GiveAbility(StartSpec);        // 어빌리티 발동 준비 완료!
	}
}

void AABGASFountain::BeginPlay()
{
	Super::BeginPlay();

	GetWorld()->GetTimerManager().SetTimer(ActionTimerHandle,
		this,							// 이 오브젝트 (AABGASFountain),
		&AABGASFountain::TimerAction,	// 실행할 메소드 바인딩
		ActionPeriod,					// 타이머 주기 지정
		true,							// 반복 (loop)
		0.0f);							// 딜레이 없도록 (바로 실행되도록)
}

void AABGASFountain::TimerAction()
{
	ABGAS_LOG(LogABGAS, Log, TEXT("Begin Timer"));

    // 태그 컨테이너 생성
	FGameplayTagContainer TargetTag(ABTAG_ACTOR_ROTATE);

	// HasMatchingGameplayTag : 태그가 있는지 체크
	// 태그가 존재 하면, 어빌리티가 발동되고 있다고 파악 가능
	// (ABTAG_ACTOR_ISROTATING 는 활성화될 때 넣어주는 태그로 ABGA_Rotate 어빌리티 생성자에서 지정했었으니까)
	if (!ASC->HasMatchingGameplayTag(ABTAG_ACTOR_ISROTATING))
	{
		ASC->TryActivateAbilitiesByTag(TargetTag);
	}
	else
	{
		ASC->CancelAbilities(&TargetTag);
	}
}

```





# 블루프린트 버전으로 작성

[![GA]({{ site.imageurl }}{{ page.imagefolder }}UnrealGABlueprint1.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGABlueprint1.png)  

Blueprint 폴더 생성 후 블루프린트 클래스 생성 클릭  

[![GA]({{ site.imageurl }}{{ page.imagefolder }}UnrealGABlueprint2.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGABlueprint2.png)  

위에서 작성했었던 ABGASFountain 클래스 상속받도록 클릭  

[![GA]({{ site.imageurl }}{{ page.imagefolder }}UnrealGABlueprint3.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGABlueprint3.png)  

생성된 블루프린트 열어서 GA 추가  

[![GA]({{ site.imageurl }}{{ page.imagefolder }}UnrealGABlueprint4.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGABlueprint4.png)  

추가 후 Compile 누르고 저장  

이후 블루프린트를 맵에 배치 후 실행해보면  
`PostInitializeComponents` 에서 `StartAbilities` 가 발동 되면서 ASC에 해당 어빌리티가 주어지고,  
타이머에 의해 발동 될 것이다.  