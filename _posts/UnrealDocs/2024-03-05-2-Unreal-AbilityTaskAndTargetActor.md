---
title:  "[정리][공격 판정 2] GAS / AbilityTask와 TargetActor"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, AnimNotify, GAS, GameAbilitySystem, GA, GameplayAbility, Trigger, AbilityTask, TargetActor]

toc: true
toc_sticky: true
 
date: 2024-03-05
last_modified_at: 2024-03-06
imagefolder: "UnrealDocs/GameplayTargetActor/"
---

<br>

- 목차
  - 애니메이션 몽타주의 노티파이를 활용하여 원하는 타이밍에 공격 판정
  - 애니메이션 노티파이가 발동되면, 판정을 위한 GA를 트리거해 발동
  - GA가 발동되면 공격 판정을 위한 <b>AT 실행</b>
  - <b>타겟 액터</b>를 활용해 물리 공격 판정 수행
  - 판정 결과를 시각적으로 확인할 수 있도록 DrawDebug 기능 사용



# AbilityTask와 TargetActor

[![GA]({{ site.imageurl }}{{ page.imagefolder }}AbilityTask_TargetActor_Flow.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealDocs/GameplayTargetActor/AbilityTask_TargetActor_Flow.png)  

어빌리티 태스크와 타겟 액터의 흐름도.  



## GA가 발동되면 공격 판정을 위한 AT 실행





### 1. AbilityTask 생성

`UAbilityTask` 를 상속 받아 생성한다.  
[AT의 제작 규칙(링크)](https://eggmong.github.io/unrealdocs/1-Unreal-GameplayAbillitySystem/#at%EC%9D%98-%EC%A0%9C%EC%9E%91-%EA%B7%9C%EC%B9%99)에 따라 클래스를 작성한다.  

- (자기 자신의) AT 인스턴스를 생성해 반환하는 static 함수 작성
- AT가 종료되면, AT를 호출한 GA에 알려줄 델리게이트를 선언하여 브로드캐스팅
- 시작과 종료에서 어떠한 처리가 필요하다면, Activate와 OnDestroy 함수를 재정의(override) 하여 구현



> <b>SetWaitingOnAvatar</b> : [AT의 제작 규칙(링크)](https://eggmong.github.io/unrealdocs/1-Unreal-GameplayAbillitySystem/#at%EC%9D%98-%EC%A0%9C%EC%9E%91-%EA%B7%9C%EC%B9%99) 참고  
> UFUNCTION(<b>BlueprintCallable</b>) : 블루프린트에서 인식해서 사용 가능  
> UPROPERTY(<b>BlueprintAssignable</b>) : 블루프린트에서 인식해서 사용 가능  
>   
> ```cpp
> // GA/AT/ABAT_JumpAndWaitForLanding.h
> UFUNCTION(BlueprintCallable, Category = "Ability|Tasks", meta = (DisplayName = "JumpAndWaitForLanding", HidePin = "OwningAbility", DefaultToSelf = "OwningAbility", BlueprintInternalUseOnly = "TRUE"))
> static UABAT_JumpAndWaitForLanding* CreateTask(UGameplayAbility* OwningAbility);
>   
> UPROPERTY(BlueprintAssignable)
> FJumpAndWaitForLandingDelegate OnComplete;
> ```
>   
> [![GA]({{ site.imageurl }}{{ page.imagefolder }}BlueprintCallableAssignable.png)]({{ site.imageurl }}{{ page.imagefolder }}BlueprintCallableAssignable.png)  



#### ABAT_Trace.h

```cpp
// 공통 부분 생략

// 이 AT를 호출한 GA에 알려줄 델리게이트 선언
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FTraceResultDelegate, const FGameplayAbilityTargetDataHandle&, TargetDataHandle);

UCLASS()
class ARENABATTLEGAS_API UABAT_Trace : public UAbilityTask
{
	GENERATED_BODY()
	
public:
	UABAT_Trace();

	// 타겟 액터 클래스(ABTA_Trace)를 인자로 받았을 때, 이 클래스 정보로 하여금 스폰 타겟 액터를 시켜주도록 명령 할 것.
	UFUNCTION(BlueprintCallable, Category = "AbilityTasks", meta = (DisplayName = "JumpAndWaitForLanding", HidePin = "OwningAbility", DefaultToSelf = "OwningAbility", BlueprintInternalUseOnly = "TRUE"))
	static UABAT_Trace* CreateTask(UGameplayAbility* OwningAbility, TSubclassOf<class AABTA_Trace> TargetActorClass);

	virtual void Activate() override;
	virtual void OnDestroy(bool AbilityEnded) override;

	// 타겟 액터(TA) 생성할 함수
	void SpawnAndInitializeTargetActor();

	// TA의 타겟팅 결과 델리게이트 구독 및 마무리할 함수
	void FinalizeTargetActor();

protected:
	void OnTargetDataReadyCallback(const FGameplayAbilityTargetDataHandle& DataHandle);

public:
	UPROPERTY(BlueprintAssignable)
	FTraceResultDelegate OnComplete;

protected:
	// CreateTask 에서 받은 타겟액터 저장하기 위한 변수들을 생성

	// 생성한 타겟 액터의 클래스 정보 저장
	UPROPERTY()
	TSubclassOf<class AABTA_Trace> TargetActorClass;

	// 타겟 액터 스폰시킨 후 그걸 저장할 액터 변수
	UPROPERTY()
	TObjectPtr<class AABTA_Trace> SpawnedTargetActor;
};
```



#### ABAT_Trace.cpp

```cpp
#include "GA/TA/ABTA_Trace.h"
#include "AbilitySystemComponent.h"

UABAT_Trace::UABAT_Trace()
{
}

UABAT_Trace* UABAT_Trace::CreateTask(UGameplayAbility* OwningAbility, TSubclassOf<AABTA_Trace> TargetActorClass)
{
	UABAT_Trace* NewTask = NewAbilityTask<UABAT_Trace>(OwningAbility);

	// 타겟 액터 클래스를 인자로 받았을 때, 어빌리티태스크(AT)에게 타겟 액터를 스폰 하도록 함
	NewTask->TargetActorClass = TargetActorClass;

	return NewTask;
}

void UABAT_Trace::Activate()
{
	Super::Activate();

	SpawnAndInitializeTargetActor();
	FinalizeTargetActor();

	// 일정 시간이 지난 후(지연시킨 다음) AT를 종료하려고
	SetWaitingOnAvatar();
}

void UABAT_Trace::OnDestroy(bool AbilityEnded)
{
	if (SpawnedTargetActor)
	{
		// Task가 종료될 때, 현재 스폰된 TargetActor가 있다면 삭제 처리
		SpawnedTargetActor->Destroy();
	}

	Super::OnDestroy(AbilityEnded);
}

void UABAT_Trace::SpawnAndInitializeTargetActor()
{
	// 타겟 액터 생성

	SpawnedTargetActor = Cast<AABTA_Trace>(Ability->GetWorld()->SpawnActorDeferred<AGameplayAbilityTargetActor>(TargetActorClass, FTransform::Identity, nullptr, nullptr, ESpawnActorCollisionHandlingMethod::AlwaysSpawn));

	if (SpawnedTargetActor)
	{
		SpawnedTargetActor->SetShowDebug(true);
		SpawnedTargetActor->TargetDataReadyDelegate.AddUObject(this, &UABAT_Trace::OnTargetDataReadyCallback);

		// TargetActor에서 브로드캐스트 하는 TargetDataReadyDelegate 델리게이트를 구독하여
		// OnTargetDataReadyCallback 함수를 바인딩 한다.
		// 그러면 브로드캐스트 될 때 OnTargetDataReadyCallback 함수가 호출될 것.
	}
}

void UABAT_Trace::FinalizeTargetActor()
{
	// 타겟 액터 등록 및 마무리(StartTargeting 지시)

	// 약한 포인터 가져오기...
	UAbilitySystemComponent* ASC = AbilitySystemComponent.Get();

	if (ASC)
	{
		// Transform을 ASC의 Avatar가 갖고 있는 Transform으로 바꿔치기할 것
		const FTransform SpawnTransform = ASC->GetAvatarActor()->GetTransform();
		SpawnedTargetActor->FinishSpawning(SpawnTransform);

		// ASC에 추가하여 타겟 액터 등록
		ASC->SpawnedTargetActors.Push(SpawnedTargetActor);

		SpawnedTargetActor->StartTargeting(Ability);

		// 레티클 사용해서 추가 시각화(공격 범위 설정) 할 필요 없으니 바로 Confirm
		// Confirm 하게 되면 ABTA_Trace의 경우 ConfirmTargetingAndContinue 함수가 호출된다.
		// 최종 타겟 데이터가 생성되어, TargetDataReadyDelegate 델리게이트를 통해 AT에게 타겟 데이터 전달
		SpawnedTargetActor->ConfirmTargeting();
	}
}

void UABAT_Trace::OnTargetDataReadyCallback(const FGameplayAbilityTargetDataHandle& DataHandle)
{
	if (ShouldBroadcastAbilityTaskDelegates())
	{
		// true 라면, FTraceResultDelegate OnComplete 를 구독한 GA에게 알려준다

		// 타겟 액터로부터 확인이 끝났을 때, 즉 브로드캐스트를 받았을 때 AbilityTask는 종료 되어야 함
		// ( 받는 브로드캐스트 : TargetDataReadyDelegate.Broadcast(DataHandle); )
		// 종료되며 브로드캐스트 하여 GA에게 타겟 정보를 넘겨준다. (ABGA_AttackHitCheck)
		OnComplete.Broadcast(DataHandle);
	}

	// 태스크 종료 처리
	EndTask();
}
```





### 2. 타겟 액터 활용 및 DrawDebug

[![GA]({{ site.imageurl }}{{ page.imagefolder }}GATA1.png)]({{ site.imageurl }}{{ page.imagefolder }}GATA1.png)  

`AGameplayAbilityTargetActor` 를 상속 받은 ABTA_Trace 클래스를 생성한다.



#### ABTA_Trace.h

```cpp
// 공통 부분 생략

UCLASS()
class ARENABATTLEGAS_API AABTA_Trace : public AGameplayAbilityTargetActor
{
	GENERATED_BODY()
	
public:
	AABTA_Trace();

	// 즉시 물리 판정을 수행하기 때문에 CancelTargeting 은 구현 X

	virtual void StartTargeting(UGameplayAbility* Ability) override;

	virtual void ConfirmTargetingAndContinue() override;

	void SetShowDebug(bool InShowDebug) { bShowDebug = InShowDebug; }

protected:
	// 타겟팅 완료 되면 물리 판정하기 위한 함수
	virtual FGameplayAbilityTargetDataHandle MakeTargetData() const;

	bool bShowDebug = false;
};
```



#### ABTA_Trace.cpp

```cpp
#include "Components/CapsuleComponent.h"
#include "Physics/ABCollision.h"
#include "DrawDebugHelpers.h"

AABTA_Trace::AABTA_Trace()
{
}

void AABTA_Trace::StartTargeting(UGameplayAbility* Ability)
{
	Super::StartTargeting(Ability);

	SourceActor = Ability->GetCurrentActorInfo()->AvatarActor.Get();
}

void AABTA_Trace::ConfirmTargetingAndContinue()
{
	// 액터가 존재 한다면, 핸들을 어빌리티 태스크(ABAT_Trace)로 전송하도록 브로드캐스트 함

	if (SourceActor)
	{
		FGameplayAbilityTargetDataHandle DataHandle = MakeTargetData();
		TargetDataReadyDelegate.Broadcast(DataHandle);

		// ABAT_Trace 가 이 델리게이트를 구독하게 될 것임.
	}
}

FGameplayAbilityTargetDataHandle AABTA_Trace::MakeTargetData() const
{
	ACharacter* Character = CastChecked<ACharacter>(SourceActor);

	FHitResult OutHitResult;
	const float AttackRange = 100.0f;
	const float AttackRadius = 50.0f;

	// false : 충돌 쿼리를 단순하게. true로 하면 더 많은 정보 수집
	// 자기 자신(Character)는 제외
	FCollisionQueryParams Params(SCENE_QUERY_STAT(AABTA_Trace), false, Character);

	const FVector Forward = Character->GetActorForwardVector();

	// 캡슐의 위치로 시작점 구성 (현재 캐릭터의 위치 + 정면 방향으로 캡슐 반지름만큼 더해준 좌표값)
	const FVector Start = Character->GetActorLocation() + (Forward * Character->GetCapsuleComponent()->GetScaledCapsuleRadius());
	const FVector End = Start + (Forward * AttackRange);

	// CCHANNEL_ABACTION 이걸로 채널 설정한 타겟만 검출되도록 함
	bool HitDetected = GetWorld()->SweepSingleByChannel(OutHitResult, Start, End, FQuat::Identity, CCHANNEL_ABACTION, FCollisionShape::MakeSphere(AttackRadius), Params);

	FGameplayAbilityTargetDataHandle DataHandle;
	if (HitDetected)
	{
		FGameplayAbilityTargetData_SingleTargetHit* TargetData = new FGameplayAbilityTargetData_SingleTargetHit(OutHitResult);
		DataHandle.Add(TargetData);
	}

#if ENABLE_DRAW_DEBUG
	if (bShowDebug)
	{
		FVector CapsuleOrigin = Start + (End - Start) * 0.5f;
		float CapsuleHalfHeight = AttackRange * 0.5f;
		FColor DrawColor = HitDetected ? FColor::Green : FColor::Red;

		// 회전행렬 적용하여 캡슐 눕힘
		DrawDebugCapsule(GetWorld(), CapsuleOrigin, CapsuleHalfHeight, AttackRadius, FRotationMatrix::MakeFromZ(Forward).ToQuat(), DrawColor, false, 5.0f);
	}
#endif

	return FGameplayAbilityTargetDataHandle();
}
```





### 3. GA 에서 AT 생성하여 사용

#### ABGA_AttackHitCheck.h

```cpp
UCLASS()
class ARENABATTLEGAS_API UABGA_AttackHitCheck : public UGameplayAbility
{
	GENERATED_BODY()
	
public:
	UABGA_AttackHitCheck();

	virtual void ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData) override;

protected:
	// AbilityTask와 TargetActor가 마무리 되고, GA에서 타겟 정보를 받는 부분 구현
	UFUNCTION()
	void OnTraceResultCallback(const FGameplayAbilityTargetDataHandle& TargetDataHandle);
};
```



#### ABGA_AttackHitCheck.cpp

```cpp
#include "AbilitySystemBlueprintLibrary.h"
#include "GA/AT/ABAT_Trace.h"
#include "GA/TA/ABTA_Trace.h"

UABGA_AttackHitCheck::UABGA_AttackHitCheck()
{
	InstancingPolicy = EGameplayAbilityInstancingPolicy::InstancedPerActor;
}

void UABGA_AttackHitCheck::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
	Super::ActivateAbility(Handle, ActorInfo, ActivationInfo, TriggerEventData);

	// 1. AT 생성 (GAS, GameAbilitySystem 의 'AT의 활용 패턴' 참고)
	UABAT_Trace* AttackTraceTask = UABAT_Trace::CreateTask(this, AABTA_Trace::StaticClass());

	// 2. AT의 종료 델리게이트 구독
	AttackTraceTask->OnComplete.AddDynamic(this, &UABGA_AttackHitCheck::OnTraceResultCallback);

	// 3. GA의 구독 설정이 완료되면 AT를 구동
	AttackTraceTask->ReadyForActivation();

	ABGAS_LOG(LogABGAS, Log, TEXT("UABGA_AttackHitCheck Begin"));
}

void UABGA_AttackHitCheck::OnTraceResultCallback(const FGameplayAbilityTargetDataHandle& TargetDataHandle)
{
	// ABAT_Trace가 끝나면 이 함수가 호출될 것.

	if (UAbilitySystemBlueprintLibrary::TargetDataHasHitResult(TargetDataHandle, 0))
	{
		FHitResult HitResult = UAbilitySystemBlueprintLibrary::GetHitResultFromTargetData(TargetDataHandle, 0);

		ABGAS_LOG(LogABGAS, Log, TEXT("Target %s Detected"), *(HitResult.GetActor()->GetName()));
	}

	bool bReplicatedEndAbility = true;
	bool bWasCancelled = true;
	EndAbility(CurrentSpecHandle, CurrentActorInfo, CurrentActivationInfo, bReplicatedEndAbility, bWasCancelled);
	// 어빌리티 종료
}
```