---
title:  "[정리] GAS, GameAbilitySystem"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, GAS, GameAbilitySystem, GA, GameplayAbility, AT, AbilityTask]

toc: true
toc_sticky: true
 
date: 2024-02-26
last_modified_at: 2024-02-26
imagefolder: "UnrealDocs/GameAbilitySystem/"
---

<br>

# GameplayAbilitySystem

액터에 기능을 추가하던 기존 방식에서  
액터에서 기능을 분리하여 GAS 로 만들어서 액터에 추가해주는 방식이라  
액터가 다양한 기능들을 추가로 가질 수 있도록 <b>확장 설계</b> 할 수 있는 장점이 있다.  

## GAS 핵심 요소

### 1. 어빌리티 시스템 컴포넌트 (AbilitySystemComponent, ASC)

- GAS 프레임워크를 관리하고 처리하는, Actor의 중앙 처리 장치
- Actor에다 생성해준다. (Actor에 단 하나만 부착할 수 있음!)
- Actor가 취할 액션인 어빌리티를 구현해주면 된다.
- Actor는 부착된 ASC를 통해 게임플레이 어빌리티를 발동시킬 수 있음
- ASC를 부착한 액터 사이에 GAS 시스템의 상호 작용이 가능해짐.
  - 어빌리티 시스템 컴포넌트


***



### 2. 게임플레이 태그 (Tag)

- 현재 프로젝트 레벨에서 액터들의 상태나 행동을 관리하는 데 사용됨
- FName으로 관리되는 경량 표식 데이터
  - 액터나 컴포넌트에 지정했던 태그와는 다른 데이터임
- 프로젝트 설정에서 별도로 게임플레이 태그를 생성하고 관리할 수 있음
  - `DefaultGameplayTags.ini` 파일에 저장됨.
- 계층 구조로 구성되어서 체계적인 관리 가능
  - Actor.Action.Rotate : 행동에 대한 태그
  - Actor.State.IsRotating : 상태에 대한 태그
- 게임플레이 태그들의 저장소 : `GameplayTagContainer`
  - HasTagExact : 컨테이너에 A.1 태그가 있는 상황에서 A로 찾으면 false
  - HasAny : 컨테이너에 A.1 태그가 있는 상황에서 A와 B로 찾으면 true
  - HasAnyExact : 컨테이너에 A.1 태그가 있는 상황에서 A와 B로 찾으면 false
  - HasAll : 컨테이너에 A.1 태그와 B.1 태그가 있는 상황에서 A와 B로 찾으면 true
  - HasAllExact : 컨테이너에 A.1 태그와 B.1 태그가 있는 상황에서 A와 B로 찾으면 false
- 게임플레이 어빌리티 시스템과 독립적으로 사용 가능

[![bt]({{ site.imageurl }}{{ page.imagefolder }}GASTag.png)]({{ site.imageurl }}{{ page.imagefolder }}GASTag.png)  

Project Setting -> GameplayTags 항목에서 태그를 생성/관리할 수 있다.  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}GASTagStructure.png)]({{ site.imageurl }}{{ page.imagefolder }}GASTagStructure.png)  

Action, State 등으로 계층 구분을 지을 수 있다.  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}GASTagini.png)]({{ site.imageurl }}{{ page.imagefolder }}GASTagini.png)  

Config -> DefaultGameplayTags.ini 파일에서 추가한 태그를 확인할 수 있다.  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}GASTagHeader.png)]({{ site.imageurl }}{{ page.imagefolder }}GASTagHeader.png)  

소스 폴더에 들어가 `메모장 만들기 -> .h 로 확장자명 변경`을 통해 헤더 파일을 생성해주고,  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}GASTagHeaderContext.png)]({{ site.imageurl }}{{ page.imagefolder }}GASTagHeaderContext.png)  

`ABGameplayTag.h` 헤더파일만 인클루드하면 게임플레이 태그에 관련된 것들을  
자동으로 모두 인클루드 하는 형태가 되도록 ABGameplayTag.h에 `GameplayTagContainer.h` 인클루드 해줌.  

이런 식으로 편리하게 사용할 수 있다.


***



### 3. 게임플레이 어빌리티 (GA)

- ASC 설정이 완료되면 GA를 설정해줘야 한다.
- ASC에 등록되어 발동시킬 수 있는 액션 명령이라고 보면 된다.
  - 공격, 마법, 특수 공격 등...


#### GA의 발동 과정

- <b>ASC에 어빌리티 등록</b> : ASC의 `GiveAbility` 함수에 발동할 GA의 타입을 전달
  - 발동할 GA 타입 정보를 게임플레이 어빌리티 스펙(`GameplayAbilitySpec`) 이라고 함.
- <b>ASC에게 어빌리티를 발동하라고 명령</b> : ASC의 `TryActivateAbility` 함수에 <u>발동할 GA의 클래스 타입을 전달</u>
  - ASC에 등록된 타입이면 GA의 인스턴스가 생성된다.
- 발동된 GA에는 <b>발동한 액터와 실행 정보가 기록됨</b>
  - SpecHandle : 발동된 어빌리티에 대한 핸들
    - ASC는 직접 어빌리티를 참조하지 않고 스펙 정보만 가지고 있음.
    - 핸들 값은 전역으로 설정되어 있으며 스펙 생성시 자동으로 1씩 증가함. 기본값은 -1
    - 어빌리티 정보 : 스펙
    - 어빌리티 인스턴스에 대한 레퍼런스 : 스펙 핸들
  - ActorInfo : 어빌리티의 소유자와 아바타 정보
  - ActivationInfo : 발동 방식에 대한 정보

> 👉 GameplayAbilitySpec  
> 입력 값을 설정하는 필드 InputID가 제공됨.  
> ASC에 등록된 스펙을 검사해서 <u>입력에 매핑된 GA를 찾을 수 있음</u> : `FindAbilitySpecFromInputID`  
> 사용자 입력이 들어오면 ASC에서 입력에 관련된 GA를 검색함  
> 
> 해당 GA를 발견하면, 현재 발동 중인지 판별  
> - GA가 발동 중이면 입력이 왔다는 신호를 전달 : `AbilitySpecInputPressed`  
> - GA가 발동하지 않았다면 새롭게 발동 시킬 수 있음 : `TryActivateAbility`  
>
> 입력이 떨어지면 동일하게 처리  
> - GA에게 입력이 떨어졌다는 신호 전달 : `AbilitySpecInputReleased`  


#### GA의 주요 함수

- CanActivateAbility : 어빌리티가 발동될 수 있는지 없는지 파악해주는 함수
- <b>ActivateAbility : 어빌리티가 발동될 때 호출</b>
- <b>CancelAbility : 어빌리티가 취소될 때 호출</b>
- EndAbility : 어빌리티를 스스로 마무리할 때 호출


#### GA의 인스턴싱 옵션

InstancingPolicy

상황에 따라 다양한 인스턴스 정책을 지정할 수 있음.  

- NonInstanced : 인스턴싱 없이 CDO 개체에서 어빌리티 일괄 처리
  - 가장 가볍지만, '상태'가 부여되는 GA를 만들고자 할 땐 부적합
- InstancedPerActor : 액터마다 하나의 어빌리티 인스턴스를 만들어 처리 (Primary Instance : 가장 먼저 생성된 인스턴스)  
  - 가장 무난함
  - '상태'를 필요로 할 때 사용 (점프 등)
  - 네트워크 리플리케이션시 정보들도 같이 네트워크를 통해서 넘어가게 되는데,  
  InstancedPerActor 의 경우 리플리케이션이 용이함.
- InstancedPerExecution : 발동할 때 마다 인스턴스를 생산함.

Abilities/GameplayAbility_CharacterJump 에서 호출 예시 확인 가능.  

```cpp
UGameplayAbility_CharacterJump::UGameplayAbility_CharacterJump(const FObjectInitializer& ObjectInitializer)
	: Super(ObjectInitializer)
{
	NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::LocalPredicted;
	InstancingPolicy = EGameplayAbilityInstancingPolicy::NonInstanced;
}
```


***



### 4. 게임플레이 이펙트 (GE)

- 특수효과가 아닌 '영향' 이라는 뜻.
- 게임플레이 어빌리티가 발동하면 행동에 대한 결과로 어떤 효과가 나타나는데,  
이 때 이 이펙트를 사용해서 어빌리티의 결과에 대한 효과를 시스템에 알려주는 형태.
   - 게임플레이 이펙트
   - 이펙트 실행 계산
   - 장식 이펙트 게임플레이 큐


***



### 5. 어트리뷰트 (Attribute)

- 이런 효과들은 스탯이나 데이터에 영향을 미치게 되는데, 이런 스탯 같은 데이터를 Attribute 라고 함.
   - 게임플레이 어트리뷰트 데이터
   - 어트리뷰트 세트


***



### 6. 어빌리티 태스크 (AbilityTask, AT)

- 게임플레이 어빌리티(GA)의 실행은 한 프레임에서 이루어짐.
- GA가 시작되면 `EndAbility` 나 `CancelAbility` 함수가 호출되기 전 까지 끝나지 않음. 계속 발동 중인 상태.
- 애니메이션 재생 같이 <b>시간이 소요</b>되고, <b>상태를 관리</b>해야 하는 어빌리티의 구현 방법이다.
  - 비동기적으로 작업을 수행하고 끝나면 결과를 통보받는 형태로 구현해야 함.
  - 이를 위해 GAS는 어빌리티 태스크를 제공한다는 것!


#### AT의 활용 패턴

1. 어빌리티 태스크에 <u>작업이 끝나면 브로드캐스팅되는 종료 델리게이트를 선언</u>함.
2. GA는 AT를 생성한 후 바로 종료 델리게이트를 구독함
3. GA의 구독 설정이 완료되면 AT를 구동 : <b>AT의 ReadyForActivation 함수 호출</b>
4. AT의 작업이 끝나면 델리게이트를 구독한 GA의 콜백 함수가 호출됨
5. GA의 콜백함수가 호출되면 <u>GA의 EndAbility 함수를 호출해 GA 종료</u>

GA는 필요에 따라 다수의 AT를 사용하여 복잡한 액션 로직을 설계할 수 있다.  

eg.  
AbilityTask_PlayMontageAndWait : 몽타주를 재생하고 끝날 때 까지 기다려주는 AT


#### AT의 제작 규칙

- AT는 UAbilityTask 클래스를 상속받아 제작한다.
- (자기 자신의) AT 인스턴스를 생성해 반환하는 `static` 함수를 선언해 구현한다. (항상 X)
- AT가 종료되면, AT를 호출한 GA에 알려줄 <b>델리게이트</b>를 선언해야 한다.
- 시작과 종료에서 어떠한 처리가 필요하다면, <b>Activate</b>와 <b>OnDestroy</b> 함수를 재정의(override) 하여 구현한다.
- 일정 시간이 지난 후 AT를 종료하고자 한다면, 활성화시 `SetWaitingOnAvatar` 함수를 호출해 <u>Waiting 상태</u>로 설정한다.
  - > 점프를 수행하고 나서 바로 끝나는 것이 아니라, 땅에 닿을 때 까지 기다려야 할 때.
- 만일 <b>Tick</b>을 활성화하고 싶다면 UAbilityTask의 멤버변수인 <b>bTickingTask</b> 값을 true로 설정한다.
- AT가 종료되면, 앞에서 선언한 델리게이트를 브로드캐스팅 한다.


#### GameplayAbility와 AbilityTask 사이의 실행 흐름

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealAT.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealAT.png)  


***



# GAS Flow (흐름)

[![bt]({{ site.imageurl }}{{ page.imagefolder }}GASFlow.png)]({{ site.imageurl }}{{ page.imagefolder }}GASFlow.png)  

모든 흐름의 중심에는 `AbilitySystemComponent` 가 있다.  
이 주위로 `Attribute`나 `Gameplay Ability`가 존재한다.  

`AbilitySystemComponent` 는 모든 요소를 직접 제어할 수 있다.  

`Gameplay Tag` 정보는 프로젝트 단위에서 모든 시스템을 감싸고 있음.  
어떤 로직을 전개할 때 이 태그를 사용해서 현재 상태를 기록하고 파악할 수 있다.  

어떤 어빌리티 액션을 발동시킬 때, 애니메이션과 같이 결합이 된다면 애니가 재생되기까지의 일정 시간이 필요할 수 있음.  
이렇게 시간이 걸리는 작업들은 `Task`라는 개념으로 단위화시켜서  
이런 Task를 조합하여 원하는 액션으로 제작할 수 있도록 설계되어 있음.  

액션의 결과로 어떤 영향이 발동되면 이 영향을 강조하기 위해 시각적or청각적 효과를 주는데,  
GAS에선 이것을 `Gameplay Cue` 라고 한다.  
(일반적으로 우리가 '이펙트'라고 부르는 바로 그 효과! fx!)




# 블루프린트에서 호출을 위한 제작 규칙

- static 함수에 UFUNCTION(BlueprintCallable) 지정
- 콜백을 위한 델리게이트는 Dynamic Delegate 로 선언한다.
- AT의 델리게이트에 UPROPERTY(BlueprintAssignable) 을 지정한다.

```cpp
UFUNCTION(BlueprintCallable, Category = "Ability|Tasks", meta = (DisplayName = "JumpAndWaitForLanding", HidePin = "OwningAbility", DefaultToSelf = "OwningAbility", BlueprintInternalUseOnly = "TRUE"))
static UABAT_JumpAndWaitForLanding* CreateTask(UGameplayAbility* OwningAbility);

// ...

UPROPERTY(BlueprintAssignable)
FJumpAndWaitForLandingDelegate OnComplete;
```

[![bt]({{ site.imageurl }}UnrealDocs/BlueprintMakeRule.png)]({{ site.imageurl }}UnrealDocs/BlueprintMakeRule.png)  