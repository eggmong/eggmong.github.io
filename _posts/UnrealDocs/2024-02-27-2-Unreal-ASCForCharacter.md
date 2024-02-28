---
title:  "[정리] GAS / 캐릭터에 ASC 설정 및 어빌리티 추가"

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

# 캐릭터에 ASC 설정 및 어빌리티 추가

전에 했었던 Fountain 과 같이 플레이어 캐릭터에 설정하는 것도 가능함.  
하지만 네트워크 플레이를 감안하면, 서버 -> 클라이언트로 배포되는 액터가 더 적합하다.  
> 서버에서 플레이어와 플레이어가 빙의하는 '폰'이 생성되면  
> 이 정보를 클라이언트에게 어떻게 넘겨줄지 고민해야 하는데,  
> GAS 시스템도 이를 감안해 서버에서 보관해야 될 데이터와 플레이어를 실제로 조종하고 있는 캐릭터를  
> 분리해서 관리하는 것이 좋다.  
> 
> GAS 프레임워크의 현재 상태는 결국 데이터로 관리되기 때문에,  
> 이 프레임워크의 데이터를 누가 주도적으로 관리할지 결정해줘야 한다.  

이 때 많이 사용하는 액터는 <u>주기적으로 플레이어 정보를 배포</u>하는 `PlayerState` 액터이다.  

하지만 실제 게임에서의 상호작용은 '폰'이 담당하기 때문에  
PlayerState의 역할과 폰의 역할을 구분해주어야 한다.  

따라서 <b>Owner를 PlayerState로 설정</b>하여 데이터를 담당하도록 하고,  
게임에서 물체와 상호작용하는 비주얼적인 부분을 담당하는 액터를 Avatar 액터라고 하는데  
<b>Avatar를 Character로 설정</b> 하는 것이 일반적인 방법이다!  

[![GA]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase1.png)]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase1.png)  

그래서 플레이어 캐릭터를 생성할 때  
GAS 프레임워크는 <b>PlayerState에서 ASC를 생성</b>하고, 이 생성된 ASC 포인터를 캐릭터에 넘겨주는 것이 바람직하다.  

발동한 캐릭터의 어빌리티들은 대부분 모두 캐릭터 중심으로 진행될 것이고  
어빌리티로 인한 결과는 ASC가 데이터로 보관하고, 이것들은 실시간으로 서버에서 클라이언트로 전송될 것이다.  

***

## 1. GameMode 세팅

[![GA]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase2.png)]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase2.png)  

GameModeBase를 상속받은 블루프린트 클래스를 만든 뒤,  
PlayerController 와 PlayerState 넣어주고, 플레이어 캐릭터도 넣어준다.  

[![GA]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase3.png)]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase3.png)  

월드 세팅에서 GameMode 넣어준다.  

[![GA]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase4.png)]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase4.png)  

프로젝트 세팅에서도 GameMode 넣어준다.  




## 2. PlayerState 생성

### ABGASPlayerState.h

```cpp
#include "AbilitySystemInterface.h"

UCLASS()
class ARENABATTLEGAS_API AABGASPlayerState : public APlayerState, public IAbilitySystemInterface
{
	GENERATED_BODY()
	
public:
	AABGASPlayerState();

	virtual class UAbilitySystemComponent* GetAbilitySystemComponent() const override;
    // ASC 인터페이스 상속받아 구현해주는 게 일반적인 방법!

protected:
	UPROPERTY(EditAnywhere, Category = GAS)
	TObjectPtr<class UAbilitySystemComponent> ASC;
};
```



### ABGASPlayerState.cpp

```cpp
#include "AbilitySystemComponent.h"

AABGASPlayerState::AABGASPlayerState()
{
	ASC = CreateDefaultSubobject<UAbilitySystemComponent>(TEXT("ASC"));

	//ASC->SetIsReplicated(true);
}

UAbilitySystemComponent* AABGASPlayerState::GetAbilitySystemComponent() const
{
	return ASC;
}
```

#### SetIsReplicated

네트워크 플레이에서 ASC는 서버에서 클라이언트로 계속 전송이 되어야 함.  
이를 위해 컴포넌트가 리플리케이션이 되도록 SetIsReplicated 를 true로 설정해야 함.  




## 3. ABGASCharacterPlayer 생성

### ABGASCharacterPlayer.h

```cpp
#include "Character/ABCharacterPlayer.h"
#include "AbilitySystemInterface.h"

UCLASS()
class ARENABATTLEGAS_API AABGASCharacterPlayer : public AABCharacterPlayer, public IAbilitySystemInterface
{
	GENERATED_BODY()
	
public:
	AABGASCharacterPlayer();

	virtual class UAbilitySystemComponent* GetAbilitySystemComponent() const override;

	// 플레이어가 빙의될 때 호출되는 함수
	// 여기서 PlayerStste에서 생성된 ASC를 받아올 것임.
	// 네트워크 플레이의 경우 서버에서만 호출되는 형태가 될 것이다.
	virtual void PossessedBy(AController* NewController) override;

	// 입력 처리를 위한 재정의
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

protected:
	void SetupGASInputComponent();
	
	// 입력 눌렀을 때
	void GASInputPressed(int32 InputId);

	// 입력이 떨어졌을 때
	void GASInputReleased(int32 InputId);

protected:
	UPROPERTY(EditAnywhere, Category = GAS)
	TObjectPtr<class UAbilitySystemComponent> ASC;

	// 입력에 대한 어빌리티 목록 생성
	UPROPERTY(EditAnywhere, Category = GAS)
	TMap<int32, TSubclassOf<class UGameplayAbility>> StartInputAbilities;
};
```


### ABGASCharacterPlayer.cpp

생성자에서 ASC를 생성해줄 때,  
캐릭터의 생성자에서 ASC를 생성하면 데이터를 관리하는 주체가 2개가 되어버린다.  
(이미 PlayerState 에서 1개 생성해놨기 때문에)

그래서 일단 null로 설정하고  
실제 플레이어가 캐릭터에 빙의할 때 PlayerState에서 생성했던 ASC 값을 넣어주는 형태로 구현한다.  

```cpp
#include "Character/ABGASCharacterPlayer.h"
#include "AbilitySystemComponent.h"
#include "Player/ABGASPlayerState.h"
#include "EnhancedInputComponent.h"

AABGASCharacterPlayer::AABGASCharacterPlayer()
{
	ASC = nullptr;

	static ConstructorHelpers::FObjectFinder<UAnimMontage> ComboActionMontageRef(TEXT("/Script/Engine.AnimMontage'/Game/ArenaBattleGAS/Animation/AM_ComboAttack.AM_ComboAttack'"));
	if (ComboActionMontageRef.Object)
	{
		ComboActionMontage = ComboActionMontageRef.Object;
	}
}

UAbilitySystemComponent* AABGASCharacterPlayer::GetAbilitySystemComponent() const
{
	return ASC;
}

void AABGASCharacterPlayer::PossessedBy(AController* NewController)
{
	Super::PossessedBy(NewController);

	AABGASPlayerState* GASPS = GetPlayerState<AABGASPlayerState>();

	if (GASPS)
	{
		ASC = GASPS->GetAbilitySystemComponent();
		ASC->InitAbilityActorInfo(GASPS, this);
		// 오너 정보에 PlayerState을, 아바타 정보엔 this(Player) 넣어주어서 ASC 초기화 완료

		for (const auto& StartInputAbility : StartInputAbilities)
		{
			FGameplayAbilitySpec StartSpec(StartInputAbility.Value);
			StartSpec.InputID = StartInputAbility.Key;
			ASC->GiveAbility(StartSpec);
		}

		SetupGASInputComponent();

		// 플레이어가 캐릭터에 빙의할 때 콘솔 커맨드를 호출하도록 함
		APlayerController* PlayerController = CastChecked<APlayerController>(NewController);
		PlayerController->ConsoleCommand(TEXT("showdebug abilitysystem"));
	}


}

void AABGASCharacterPlayer::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	SetupGASInputComponent();
}

void AABGASCharacterPlayer::SetupGASInputComponent()
{
	if (IsValid(ASC) && IsValid(InputComponent))
	{
		UEnhancedInputComponent* EnhancedInputComponent = CastChecked<UEnhancedInputComponent>(InputComponent);

		// 어떤 입력이 들어왔는지 파악해야 입력에 따른 어빌리티를 실행할 수 있음
		// EnhancedInputComponent 경우엔 추가적인 데이터를 넘겨줄 수 있도록 설계되어 있음
		// (바인드 할 함수의 매개변수로 넘김)
		EnhancedInputComponent->BindAction(JumpAction, ETriggerEvent::Triggered, this, &AABGASCharacterPlayer::GASInputPressed, 0);
		EnhancedInputComponent->BindAction(JumpAction, ETriggerEvent::Completed, this, &AABGASCharacterPlayer::GASInputReleased, 0);
		
		EnhancedInputComponent->BindAction(AttackAction, ETriggerEvent::Triggered, this, &AABGASCharacterPlayer::GASInputPressed, 1);
	}
}

void AABGASCharacterPlayer::GASInputPressed(int32 InputId)
{
	FGameplayAbilitySpec* Spec = ASC->FindAbilitySpecFromInputID(InputId);
	if (Spec)
	{
		Spec->InputPressed = true;		// 스펙에 현재 상태 지정
		if (Spec->IsActive())
		{
			// 어빌리티가 현재 발동이 되어 있는 상태라면
			
			ASC->AbilitySpecInputPressed(*Spec);
			// 입력이 왔다는 신호를 전달

            // ABGA_Attack 의 InputPressed 함수 호출 될 것임.
		}
		else
		{
			// 발동하지 않았다면 새롭게 발동 시킴
			ASC->TryActivateAbility(Spec->Handle);
		}
	}
}

void AABGASCharacterPlayer::GASInputReleased(int32 InputId)
{
	FGameplayAbilitySpec* Spec = ASC->FindAbilitySpecFromInputID(InputId);
	if (Spec)
	{
		Spec->InputPressed = false;
		if (Spec->IsActive())
		{
			ASC->AbilitySpecInputReleased(*Spec);
		}
	}
}
```

#### 캐릭터 BP에 StartInputAbilities 추가

[![GA]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase6.png)]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase6.png)  





# GA 블루프린트화 하여 GA 실행될 때 GameplayTag 바뀌도록 설정

[![GA]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase_blueprint1.png)]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase_blueprint1.png)  

GA 선택하여 블루프린트 클래스 생성  

[![GA]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase_blueprint2.png)]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase_blueprint2.png)  

ActivationOwnedTags 에 태그 추가  
(활성화 될 때 발동되는 태그)  

[![GA]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase_blueprint3.png)]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase_blueprint3.png)  

캐릭터 블루프린트의 GAS 항목에 블루프린트로 만든 어빌리티로 교체  




# 코드에서 콘솔 커맨드 출력

```cpp
// void AABGASCharacterPlayer::PossessedBy(AController* NewController) 에서

// 플레이어가 캐릭터에 빙의할 때 콘솔 커맨드를 호출하도록 함
APlayerController* PlayerController = CastChecked<APlayerController>(NewController);
PlayerController->ConsoleCommand(TEXT("showdebug abilitysystem"));
```