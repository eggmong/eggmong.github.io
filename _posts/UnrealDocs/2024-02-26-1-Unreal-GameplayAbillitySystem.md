---
title:  "[정리] GAS, GameAbilitySystem"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, GAS, GameAbilitySystem]

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
  - ActorInfo : 어빌리티의 소유자와 아바타 정보
  - ActivationInfo : 발동 방식에 대한 정보

#### GA의 주요 함수

- CanActivateAbility : 어빌리티가 발동될 수 있는지 없는지 파악해주는 함수
- <b>ActivateAbility : 어빌리티가 발동될 때 호출</b>
- <b>CancelAbility : 어빌리티가 취소될 때 호출</b>
- EndAbility : 어빌리티를 스스로 마무리할 때 호출



### 4. 게임플레이 이펙트 (GE)

- 특수효과가 아닌 '영향' 이라는 뜻.
- 게임플레이 어빌리티가 발동하면 행동에 대한 결과로 어떤 효과가 나타나는데,  
이 때 이 이펙트를 사용해서 어빌리티의 결과에 대한 효과를 시스템에 알려주는 형태.
   - 게임플레이 이펙트
   - 이펙트 실행 계산
   - 장식 이펙트 게임플레이 큐



### 5. 어트리뷰트 (Attribute)

- 이런 효과들은 스탯이나 데이터에 영향을 미치게 되는데, 이런 스탯 같은 데이터를 Attribute 라고 함.
   - 게임플레이 어트리뷰트 데이터
   - 어트리뷰트 세트


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

