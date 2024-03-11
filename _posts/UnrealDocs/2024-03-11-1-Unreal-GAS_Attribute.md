---
title:  "[정리] GAS / 어트리뷰트 (Attribute)"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, GAS, GameAbilitySystem, AttributeSet]

toc: true
toc_sticky: true
 
date: 2024-03-11
last_modified_at: 2024-03-11
imagefolder: "UnrealDocs/Attribute/"
---

<br>

# 어트리뷰트 세트 (Attribute Set)

- 단일 어트리뷰트 데이터인 <b>GameplayAttributeData</b>의 묶음을 어트리뷰트 '세트'라고 한다.
- GameplayAttributeData는 하나의 값이 아닌 두 가지 값으로 구성되어 있다.
  - BaseValue : 기본 값. 영구히 적용되는 고정 스탯 값을 관리하는데에 사용함.
  - CurrentValue : 변동 값. 버프스킬 등으로 <u>임시적으로 변동된 값</u>을 관리하는데에 사용함.
- 어트리뷰트 세트의 주요 함수 (오버라이드하여 사용할 수 있도록 제공함)
  - PreAttributeChange : 어트리뷰트 <b>변경 전</b>에 호출  
  변경될 값을 레퍼런스(&)로 넘겨줘서, 이 값을 변경해도 되는지 체크하여 값을 바꾸거나 냅두도록 사용할 수 있음.  
  - PostAttributeChange : 어트리뷰트 <b>변경 후</b>에 호출  
  변경 되기 전의 값과 변경 후의 값을 인자로 넘겨줘서 변화가 일어났다는 걸 알려줄 수 있음. log남길 때 씀.
  - <b>PreGameplayEffectExecute</b> : 게임플레이 이펙트 적용 전에 호출
  - <b>PostGameplayEffectExecute</b> : 게임플레이 이펙트 적용 후에 호출
- 어트리뷰트 세트 접근자 매크로
  - 많이 수행되는 기능에 대해 매크로를 만들어 제공
- <b>ASC는 초기화될 때, 같은 액터에 있는(OwnerActor) `UAttributeSet` 타입 객체를 찾아서 '자동으로' 등록한다.</b>



## 매크로

`AttributeSet.h` 에 들어가보면,  

```cpp
 * To use this in your game you can define something like this, and then add game-specific functions as necessary:
 * 
 *	#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
 *	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
 *	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
 *	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
 *	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
```
이렇게 매크로에 대한 설명이 있다. 생성한 어트리뷰트 세트 클래스에 복붙하여 사용할 수 있다.  
`ATTRIBUTE_ACCESSORS` 라는 매크로는 밑에 작성된 4개의 함수,  
즉 <b>특정 속성에 대해서 총 4개의 함수를 이 매크로 하나로 지정해준다</b> 는 것이다.  

- GAMEPLAYATTRIBUTE_PROPERTY_GETTER
  - 어트리뷰트를 나타내는 클래스에 등록된 프로퍼티를 가져온다.
  - 어트리뷰트를 처리할 때 이게 해당 어트리뷰트가 맞는지 비교할 때 사용할 수 있음.
- GAMEPLAYATTRIBUTE_VALUE_GETTER
  - CurrentValue 값을 가져오는 매크로
- GAMEPLAYATTRIBUTE_VALUE_SETTER
  - BaseValue 값을 변경해주는 매크로... BaseValue 값은 영구값이라서, 즉 내 스탯을 '확정'해주는 용도.
- GAMEPLAYATTRIBUTE_VALUE_INITTER
  - BaseValue와 CurrentValue를 같은 값으로 지정해주는 역할.

```cpp
// ABCharacterAttributeSet.h

#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)

// ...

class ARENABATTLEGAS_API UABCharacterAttributeSet : public UAttributeSet
{
	GENERATED_BODY()
	
public:
	UABCharacterAttributeSet();

	ATTRIBUTE_ACCESSORS(UABCharacterAttributeSet, AttackRange);
	

protected:
	UPROPERTY(BlueprintReadOnly, Category="Attack", Meta = (AllowPrivateAccess = true))
	FGameplayAttributeData AttackRange;
}
```

```cpp
// ABCharacterAttributeSet.cpp

// 생성자에서 초기값 이니셜라이저로 초기화 해주기!
UABCharacterAttributeSet::UABCharacterAttributeSet() :
	AttackRange(100.0f)
{
}
```



##  ASC에서 Attribute를 추가하는 동작 원리

`AbilitySystemComponent` 클래스에 들어가보면,  
`InitializeComponent` 라는 함수가 존재한다.  



```cpp
// in AbilitySystemComponent class

void UAbilitySystemComponent::InitializeComponent()
{
	Super::InitializeComponent();

	AActor *Owner = GetOwner();
	InitAbilityActorInfo(Owner, Owner);	// Default init to our outer owner

	// cleanup any bad data that may have gotten into SpawnedAttributes
	for (int32 Idx = SpawnedAttributes.Num()-1; Idx >= 0; --Idx)
	{
		if (SpawnedAttributes[Idx] == nullptr)
		{
			SpawnedAttributes.RemoveAt(Idx);
		}
	}

	TArray<UObject*> ChildObjects;
	GetObjectsWithOuter(Owner, ChildObjects, false, RF_NoFlags, EInternalObjectFlags::Garbage);

	for (UObject* Obj : ChildObjects)
	{
		UAttributeSet* Set = Cast<UAttributeSet>(Obj);
		if (Set)  
		{
			SpawnedAttributes.AddUnique(Set);
			bIsNetDirty = true;
		}
	}

	SetSpawnedAttributesListDirty();
}
```

Owner가 가지고 있는 자식 오브젝트들을 TArray로 가져온 다음에  
for문으로 돌려서 자식들 중에 타입이 `UAttributeSet`인게 있다면  
`SpawnedAttributes` 라고 하는 어트리뷰트 목록에 AddUnique로 추가를 해준다.  

> 💡 SpawnedAttributes이 TArray로 선언되었기에 중복된 어트리뷰트셋을 2개 선언할 수 <b>없다.</b>  
> 다만, '타입'이 다르면 여러 개를 가질 수 있다.  





# 코드 예시

## Player & NPC에 어트리뷰트 설정





### 1. AttributeSet 생성

AttributeSet를 상속 받은 클래스를 생성한다.  

생성된 클래스에 `AbilitySystemComponent.h` 를 include하고, 필요한 어트리뷰트 데이터들을 작성한다.  
필요할 경우 `PreAttributeChange` 나 `PostAttributeChange` 함수도 오버라이드 하여 작성해준다.  

PreAttributeChange 의 경우 어트리뷰트 값 변경 전에 호출되어  
변경될 값을 레퍼런스(&)로 넘겨줘서, 이 값을 변경해도 되는지 파악하여 값을 바꾸거나 냅둘 수 있으므로  
구현해주어야 할 것이다.  

그리고 헤더에 `AbilitySystemComponent.h` 를 인클루드하고  
UAttributeSet 클래스에 있는 매크로를 복붙한 뒤 작성해준다.  

매크로를 사용하는 이유는, 어트리뷰트를 만들면 이것에 접근할 Get, 변경할 Set 등  
여러 함수가 필요한데, 매크로를 선언해줌으로써 코드를 줄일 수 있다.  



#### ABCharacterAttributeSet.h

```cpp
#include "AbilitySystemComponent.h"


#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)

UCLASS()
class ARENABATTLEGAS_API UABCharacterAttributeSet : public UAttributeSet
{
	GENERATED_BODY()
	
public:
	UABCharacterAttributeSet();

	ATTRIBUTE_ACCESSORS(UABCharacterAttributeSet, AttackRange);
	ATTRIBUTE_ACCESSORS(UABCharacterAttributeSet, MaxAttackRange);
	ATTRIBUTE_ACCESSORS(UABCharacterAttributeSet, AttackRadius);
	ATTRIBUTE_ACCESSORS(UABCharacterAttributeSet, MaxAttackRadius);
	ATTRIBUTE_ACCESSORS(UABCharacterAttributeSet, AttackRate);
	ATTRIBUTE_ACCESSORS(UABCharacterAttributeSet, MaxAttackRate);
	ATTRIBUTE_ACCESSORS(UABCharacterAttributeSet, Health);
	ATTRIBUTE_ACCESSORS(UABCharacterAttributeSet, MaxHealth);
	

protected:
	UPROPERTY(BlueprintReadOnly, Category="Attack", Meta = (AllowPrivateAccess = true))
	FGameplayAttributeData AttackRange;

	UPROPERTY(BlueprintReadOnly, Category = "Attack", Meta = (AllowPrivateAccess = true))
	FGameplayAttributeData MaxAttackRange;

	UPROPERTY(BlueprintReadOnly, Category = "Attack", Meta = (AllowPrivateAccess = true))
	FGameplayAttributeData AttackRadius;

	UPROPERTY(BlueprintReadOnly, Category = "Attack", Meta = (AllowPrivateAccess = true))
	FGameplayAttributeData MaxAttackRadius;

	UPROPERTY(BlueprintReadOnly, Category = "Attack", Meta = (AllowPrivateAccess = true))
	FGameplayAttributeData AttackRate;

	UPROPERTY(BlueprintReadOnly, Category = "Attack", Meta = (AllowPrivateAccess = true))
	FGameplayAttributeData MaxAttackRate;

	UPROPERTY(BlueprintReadOnly, Category = "Health", Meta = (AllowPrivateAccess = true))
	FGameplayAttributeData Health;

	UPROPERTY(BlueprintReadOnly, Category = "Health", Meta = (AllowPrivateAccess = true))
	FGameplayAttributeData MaxHealth;

protected:
	virtual void PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue) override;

	virtual void PostAttributeChange(const FGameplayAttribute& Attribute, float OldValue, float NewValue) override;
};
```



#### ABCharacterAttributeSet.cpp

```cpp
#include "ArenaBattleGAS.h"

UABCharacterAttributeSet::UABCharacterAttributeSet() :
	AttackRange(100.0f),
	AttackRadius(50.f),
	AttackRate(30.0f),
	MaxAttackRange(300.0f),
	MaxAttackRadius(150.0f),
	MaxAttackRate(100.0f),
	MaxHealth(100.0f)
{
	InitHealth(GetMaxHealth());
}

void UABCharacterAttributeSet::PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue)
{
	if (Attribute == GetHealthAttribute())
	{
		// 어트리뷰트가 현재 hp어트리뷰트일 때 hp값을 최소 0 혹은 최댓값으로 숫자의 범위를 지정
		NewValue = FMath::Clamp(NewValue, 0.0f, GetMaxHealth());
	}
}

void UABCharacterAttributeSet::PostAttributeChange(const FGameplayAttribute& Attribute, float OldValue, float NewValue)
{
	if (Attribute == GetHealthAttribute())
	{
		ABGAS_LOG(LogABGAS, Log, TEXT("Health : %f -> %f"), OldValue, NewValue);
	}
}
```





### 2. Player에 어트리뷰트 추가

Player의 경우 ASC를 PlayerStat 에 추가했었다.  
그러므로 어트리뷰트도 PlayerStat에 추가한다.  



#### ABGASPlayerStat.h

```cpp
#include "AbilitySystemInterface.h"

UCLASS()
class ARENABATTLEGAS_API AABGASPlayerState : public APlayerState, public IAbilitySystemInterface
{
	GENERATED_BODY()
	
	// ...

	UPROPERTY()
	TObjectPtr<class UABCharacterAttributeSet> AttributeSet;
};
```



#### ABGASPlayerStat.cpp

```cpp
#include "Attribute/ABCharacterAttributeSet.h"

AABGASPlayerState::AABGASPlayerState()
{
	ASC = CreateDefaultSubobject<UAbilitySystemComponent>(TEXT("ASC"));

	AttributeSet = CreateDefaultSubobject<UABCharacterAttributeSet>(TEXT("AttributeSet"));
}
```




### 3. Player의 공격탐지에 어트리뷰트 사용

기존에 하드코딩 되어있던 부분(`const float AttackRange = 100.0f;`)을  
어트리뷰트에 넣은 값으로 대체해주었다.  



#### ABTA_Trace.cpp

```cpp

FGameplayAbilityTargetDataHandle AABTA_Trace::MakeTargetData() const
{
	ACharacter* Character = CastChecked<ACharacter>(SourceActor);

	// ASC 포인터를 가져온다
	UAbilitySystemComponent* ASC = UAbilitySystemBlueprintLibrary::GetAbilitySystemComponent(SourceActor);
	if (!ASC)
	{
		ABGAS_LOG(LogABGAS, Error, TEXT("ASC not found!"));
		return FGameplayAbilityTargetDataHandle();
	}

	// 어트리뷰트셋 포인터를 가져온다
	const UABCharacterAttributeSet* AttributeSet = ASC->GetSet<UABCharacterAttributeSet>();
	if (!AttributeSet)
	{
		ABGAS_LOG(LogABGAS, Error, TEXT("AttributeSet not found!"));
		return FGameplayAbilityTargetDataHandle();
	}

	FHitResult OutHitResult;
	const float AttackRange = AttributeSet->GetAttackRange();
	const float AttackRadius = AttributeSet->GetAttackRadius();

	// ...
}
```





### 4. Player의 공격 데미지를 NPC에 전달

공격 탐지에서 타겟이 있을 경우  
각각의 SourceASC(Player)와 TargetASC(NPC) 에서 Attribute를 가져와서  
SourceAttribute의 데미지 값을 TargetAttribute의 `SetHealth` 함수를 사용하여 hp를 변경함으로써 데미지를 전달한다.  

이때 SourceASC는 `GetAbilitySystemComponentFromActorInfo_Checked` 함수를 호출하여 어빌리티를 발동한 액터의 ASC를 가져올 수 있다.  

TargetASC는 `UAbilitySystemBlueprintLibrary::GetAbilitySystemComponent(HitResult.GetActor());` 함수를 호출하여  
탐지 결과에 들어있는 액터의 ASC를 가져올 수 있다. (`#include "AbilitySystemBlueprintLibrary.h"` 를 인클루드 해야한다.)  

그리고 TargetAttribute를 사용하여 Target의 Health값을 변경해야 하는데,  
GetSet 함수는 const로 반환하므로 값을 변경할 수 없다.  
그래서 <b>const_cast</b>를 사용하여 const를 떼준다.  




#### ABGA_AttackHitCheck.cpp

```cpp
void UABGA_AttackHitCheck::OnTraceResultCallback(const FGameplayAbilityTargetDataHandle& TargetDataHandle)
{
	// ABAT_Trace가 끝나면 이 함수가 호출될 것.

	if (UAbilitySystemBlueprintLibrary::TargetDataHasHitResult(TargetDataHandle, 0))
	{
		FHitResult HitResult = UAbilitySystemBlueprintLibrary::GetHitResultFromTargetData(TargetDataHandle, 0);
		ABGAS_LOG(LogABGAS, Log, TEXT("Target %s Detected"), *(HitResult.GetActor()->GetName()));

		UAbilitySystemComponent* SourceASC = GetAbilitySystemComponentFromActorInfo_Checked();

		// 데미지 전달 (타겟 액터에게)
		UAbilitySystemComponent* TargetASC = UAbilitySystemBlueprintLibrary::GetAbilitySystemComponent(HitResult.GetActor());

		if (!SourceASC || !TargetASC)
		{
			ABGAS_LOG(LogABGAS, Error, TEXT("ASC not found!"));
			return;
		}

		const UABCharacterAttributeSet* SourceAttribute = SourceASC->GetSet<UABCharacterAttributeSet>();
		UABCharacterAttributeSet* TargetAttribute = const_cast<UABCharacterAttributeSet*>(TargetASC->GetSet<UABCharacterAttributeSet>());
		// Source는 데미지값을 읽어들이고, Target은 값을 '변경'해줘야 한다.
		// 그런데 GetSet 함수는 const로 반환하므로 변경을 할 수 없다.
		// 그래서 const_cast를 하여 const를 제거한다.

		if (!SourceAttribute || !TargetAttribute)
		{
			ABGAS_LOG(LogABGAS, Error, TEXT("Attribute not found!"));
			return;
		}

		const float AttackDamage = SourceAttribute->GetAttackRate();
		TargetAttribute->SetHealth(TargetAttribute->GetHealth() - AttackDamage);
	}

	bool bReplicatedEndAbility = true;
	bool bWasCancelled = true;
	EndAbility(CurrentSpecHandle, CurrentActorInfo, CurrentActivationInfo, bReplicatedEndAbility, bWasCancelled);
	// 어빌리티 종료
}
```





### 5. NPC에 어트리뷰트 추가


#### ABGAS_CharacterNonPlayer.h

```cpp
#include "AbilitySystemInterface.h"

UCLASS()
class ARENABATTLEGAS_API AABGAS_CharacterNonPlayer : public AABCharacterNonPlayer, public IAbilitySystemInterface
{
	GENERATED_BODY()
	
public:
	AABGAS_CharacterNonPlayer();

	virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;

	// AI 컨트롤러가 이 NPC 컨트롤러를 장악할 때 ASC를 초기화 하기 위해 이 함수 내부에 구현해줄 것임.
	virtual void PossessedBy(AController* NewController) override;
	
protected:
	UPROPERTY(EditAnywhere, Category = GAS)
	TObjectPtr<class UAbilitySystemComponent> ASC;

	UPROPERTY()
	TObjectPtr<class UABCharacterAttributeSet> AttributeSet;
};
```



#### ABGAS_CharacterNonPlayer.cpp

```cpp
#include "AbilitySystemComponent.h"
#include "Attribute/ABCharacterAttributeSet.h"

AABGAS_CharacterNonPlayer::AABGAS_CharacterNonPlayer()
{
	ASC = CreateDefaultSubobject<UAbilitySystemComponent>(TEXT("ASC"));

	AttributeSet = CreateDefaultSubobject<UABCharacterAttributeSet>(TEXT("AttributeSet"));
}

UAbilitySystemComponent* AABGAS_CharacterNonPlayer::GetAbilitySystemComponent() const
{
	return ASC;
}

void AABGAS_CharacterNonPlayer::PossessedBy(AController* NewController)
{
	Super::PossessedBy(NewController);

	// npc는 PlayerStat 을 사용하지 않아서 모두 자신(this)를 넣어주게됨.
	ASC->InitAbilityActorInfo(this, this);
}
```