---
title:  "[정리][공격 판정 1] GAS / AnimNotify와 GameplayAbility Trigger"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, AnimNotify, GAS, GameAbilitySystem, GA, GameplayAbility, Trigger]

toc: true
toc_sticky: true
 
date: 2024-03-05
last_modified_at: 2024-03-06
imagefolder: "UnrealDocs/AnimNotifyAndGA/"
---

<br>

- 목차
  - 애니메이션 몽타주의 노티파이를 활용하여 원하는 타이밍에 공격 판정
  - 애니메이션 노티파이가 발동되면, 판정을 위한 GA를 트리거해 발동
  - GA가 발동되면 공격 판정을 위한 AT 실행
  - GAS에서 제공하는 타겟 액터를 활용해 물리 공격 판정 수행
  - 판정 결과를 시각적으로 확인할 수 있도록 DrawDebug 기능 사용



# AnimNotify와 GameplayAbility Trigger

## Animation Montage의 AnimNotify를 활용하여 공격 판정

### 1. AnimNotify, GameplayTags 생성 및 등록

[![GA]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA1.png)]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA1.png)  

AnimMontage에 등록할 AnimNotify를 생성한다.  
(ArenaBattleGAS 경로에 생성해주었다.)  

> 💡 AnimNotify 로드 이슈  
> 기존에 AnimMontage 를 로드할 땐, `AABCharacterBase` 의 생성자에서 로드했었다.  
> ```cpp
> static ConstructorHelpers::FObjectFinder<UAnimMontage> ComboActionMontageRef(TEXT("/Script/Engine.AnimMontage'/Game/ArenaBattle/Animation/AM_ComboAttack.AM_ComboAttack'"));
	if (ComboActionMontageRef.Object)
	{
		ComboActionMontage = ComboActionMontageRef.Object;
	}
> ```
> `AABCharacterBase` 는 `ArenaBattle` 모듈에서 생성된 클래스이다. (Source/ArenaBattle 경로에 있음)  
> 그런데 방금 위에서 생성한 AnimNotify는 `ArenaBattleGAS` 모듈에서 생성한 것이다.  
> 프로젝트의 uproject (UnrealGAS.uproject) 를 열어보면 `ArenaBattle` 모듈은 LoadingPhase가 "Default" 로 제일 먼저 로딩 된다.  
> 이후에 `ArenaBattleGAS` 모듈이 로딩 되는 순서이다.  
> 그래서 저대로 'AABCharacterBase' 에서 애님몽타주를 로드하도록 두면,  
> 애님몽타주에 등록한(할) AnimNotify는 ArenaBattleGAS 모듈에 있기 때문에  
> <u>이에 대한 정보를 찾을 수 없는 문제가 생긴다.</u>  
> 이 문제를 해결하기 위해 `ABGASCharacterPlayer` 에서 애님몽타주를 로드하도록 변경해야 한다.  



[![GA]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA2.png)]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA2.png)  

AnimNotify에 등록할 태그를 생성한다.  



[![GA]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA3.png)]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA3.png)  

AnimMontage를 열어서 기존에 등록되어 있던 AnimNotify를  
위에서 생성했던 AnimNotify로 변경한다.  
(우클릭 -> Replace With Notify... 선택하면 된다.)  



[![GA]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA4.png)]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA4.png)  

AnimNotify 눌러서 `Trigger Gameplay Tags` 항목에 위에서 생성한 태그를 AnimNotify에 등록해준다.  
<u>태그를 이용해서 GA를 발동시킬 것이다.</u>  



### 2. 발동될 GameplayAbility 클래스 및 블루프린트 생성

[![GA]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA5.png)]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA5.png)  

GA 생성하는 법은 `GameplayAbility` 상속 받는 클래스로 생성 하고 (ABGA_AttackHitCheck 로 이름 지음),  
블루프린트 클래스 생성 메뉴 눌러서 ABGA_AttackHitCheck 상속 받는 BP 생성 하면 된다.  

[![GA]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA6.png)]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA6.png)  

캐릭터 블루프린트에 생성한 BPGA를 등록해준다.  
(Input에 관련된 GA가 아니라서 StartAbilities 에 등록했다.)  



## 애니메이션 노티파이가 발동되면, 판정을 위한 GA를 트리거해 발동

### 3. GA를 트리거하여 발동

[![GA]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA7.png)]({{ site.imageurl }}{{ page.imagefolder }}AnimNotifyAndGA7.png)  

BPGA의 `Triggers` 항목의 Ability Triggers에 태그를 추가해준다.  

이렇게 하면 ASC가 등록한 어빌리티 중에서 (ABGASCharacterPlayer 의 PossessedBy 보면 `GiveAbility` 로 등록해주고 있음)  
해당 트리거가 설정된 GA를 시스템에서 자동으로 활성화 시켜준다.  

```cpp
UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(OwnerActor, TriggerGameplayTag, PayloadData);
```

AnimNotify의 Notify부분에 작성한 함수이다.  
이 함수를 호출해주면 ASC를 가지고 있는 OwnerActor 에 TriggerGameplayTag 를 넣어 이벤트를 발동시킬 수 있다.  
이걸로 GA 를 발동시킬 수 있다.  
		

- 새로운 GA가 발동되면 공격 판정을 위한 AT 실행
	- 

- GAS에서 제공하는 타겟 액터를 활용해 물리 공격 판정 수행

- 판정 결과를 시각적으로 확인할 수 있도록 DrawDebug 기능 사용

