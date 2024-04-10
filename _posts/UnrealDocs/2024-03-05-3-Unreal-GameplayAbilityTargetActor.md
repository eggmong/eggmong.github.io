---
title:  "[정리] GAS / 게임플레이 어빌리티 타겟 액터 (GameplayAbilityTargetActor)"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, GAS, GameAbilitySystem, GA, GameplayAbility]

toc: true
toc_sticky: true
 
date: 2024-03-05
last_modified_at: 2024-03-06
imagefolder: "UnrealDocs/GameplayTargetActor/"
---

<br>

# 게임플레이 어빌리티 타겟 액터

게임플레이 어빌리티(GA)에서 대상에 대한 판정(주로 물리 판정)을 구현할 때 사용하는 특수한 액터.  
줄여서 TA라고 한다.  

`AGameplayAbilityTargetActor` 클래스를 상속받아서 구현한다.  



## 타겟 액터가 필요한 이유?

타겟을 설정하는 다양한 방법이 있음.

- Trace를 사용해 즉각적으로 타겟을 검출하는 방법 (raycast 처럼...)
- 사용자의 최종 확인을 한번 더 거치는 방법 (원거리 범위 공격, 광역 공격기 혹은 장판 까는 공격 같은)
- 공격 범위 확인을 위한 추가 시각화 (시각화를 수행하는 액터를 <b>월드레티클(WorldReticle)</b>이라고 함)



### 주요 함수

- StartTargeting : 타겟팅을 시작
- ConfirmTargetingAndCoroutine : 타겟팅을 확정하고, 이후 남은 프로세스(AbilityTask) 진행
- ConfirmTargeting : 태스크 진행 없이 <b>타겟팅만 확정</b>
- CancelTargeting : 타겟팅 취소



# 게임플레이 어빌리티 타겟 데이터

위의 타겟 액터에서 판정한 <b>결과</b>를 담은 데이터.  

- Trace 히트 결과 (HitResult) 또는 SphereOverlap 의 결과
- 판정된 다수의 액터 포인터
- 시작 지점
- 끝 지점

이러한 속성을 가지고 있다.  
타겟 데이터를 여러 개 묶어 전송하는 것이 일반적인데, 이를 <b>타겟 데이터 핸들</b>이라고 함.  

타겟 액터에서 판정한 결과는 타겟 데이터 핸들을 통해서  
어빌리티 태스크에 전송된다.  



## AT와 TA사이의 실행 흐름

[![GA]({{ site.imageurl }}{{ page.imagefolder }}AbilityTask_TargetActor_Flow.png)]({{ site.imageurl }}{{ page.imagefolder }}AbilityTask_TargetActor_Flow.png)  

- SpawnActorDeferred : 타겟 액터 지연 생성