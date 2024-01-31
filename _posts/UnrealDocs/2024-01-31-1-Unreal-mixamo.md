---
title:  "기존 프로젝트에 믹사모 애니메이션 임포트하기 (IK_Retargeter 사용)"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, mixamo, animation]

toc: true
toc_sticky: true
 
date: 2024-01-31
last_modified_at: 2024-01-31
---

<br>

기존 프로젝트에 새로운 애니메이션을 추가하고 싶었다.  

[https://www.mixamo.com/#/](https://www.mixamo.com/#/)

여러 캐릭터와 애니메이션을 무료로 다운 받을 수 있다.  
adobe에서 운영하는 거라 어도비 아이디가 있다면 로그인 가능. 구글도 가능.  



# 1. mixamo 에서 fbx 다운 

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_down_first.png)  

대충 모델을 선택 후 원하는 애니메이션을 다운 받는다.  
나는 `X Bot` 이라는 모델 선택 후 `Can Can Dance` 애니메이션을 다운 받았음.  
npc가 대기 중일 땐 캉캉댄스를 추도록 바꿀 것이다ㅋ  


![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_down.png)  

애니메이션을 모델과 같이 다운받아야 하므로 with skin 을 체크해준다.  



# 2. fbx 임포트

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_import.png)  

다운 받은 fbx는 에디터에서 Mixamo 폴더를 생성하여 임포트 해준다.  
그러면 임포트 옵션 창이 뜬다.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_import_option.png)  

[https://docs.unrealengine.com/4.27/ko/WorkingWithContent/Importing/FBX/ImportOptions/](https://docs.unrealengine.com/4.27/ko/WorkingWithContent/Importing/FBX/ImportOptions/)

위에 첨부한 임포트 옵션에 대해 공식 문서를 보면,  

> 💡 Convert Scene Unit : 씬 유닛 변환 - 씬을 FBX 측정 유닛에서 UE4 측정 유닛인 센티미터로 변환합니다.  

이해는 안되지만 대충 언리얼 단위계로 바꾼다는 뜻인 것 같다.  
체크해준다.  
그리고 실질적으로 난 애니메이션이 필요하므로 import animation 옵션도 체크한다.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_import_done.png)  

그럼 임포트가 완료된다.  



# 3. IK_Rig 생성

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_rig_create.png)  

다운 받은 애니메이션엔 rig 이 없으므로 생성해줘야 한다.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_rig_select_skeleton.png)  

skeletion mesh를 선택해야 하는데, 우린 모델을 같이 다운 받았으므로 스켈레톤 메쉬가 있음.  
클릭하여 생성해준다.  

IK_rig에 대해 더 자세한 내용은 [공식 문서(링크)](https://docs.unrealengine.com/5.0/ko/ik-rig-in-unreal-engine/)를 참조한다.

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_cancan.png)  

IK_Cancan 이라고 이름 바꿔줬다.  



# 4. 생성한 IK_Rig 세부 설정

기존 프로젝트에서 내가 사용하고 있던 `IK_Warrior` 을 보면,  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_target.png)  

Retarget Chain 도 설정되어 있고, Retarget Root등 여러 항목이 설정되어 있는 걸 볼 수 있다.

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_source.png)  

하지만 방금 만든 `IK_Cancan` 은 깨-끗하다...  
직접 설정해줘야 한다.  



## 4-1. Retarget Root 설정

먼저 Retarget Root 를 설정한다.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_source_1.png)  

`IK_Warrior` 의 Retarget Root 는 `pelvis`, 골반으로 되어 있다.  
`IK_Cancan` 를 열어보면 pelvis 는 없지만 `Hips` 가 있다. 이걸로 설정한다.  

## 4-2. Retarget Chain 설정

### No Goal

그리고 Retarget Chain 을 설정해준다.  
`IK_Warrior` 에도 `Spine` 항목이 있으므로, `IK_Cancan` 에도 똑같이 해준다.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_source_2.png)  

키보드의 SHIFT 를 눌러 Spine 항목을 다 잡은 뒤,

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_source_3.png)  

`Add New Chain` 을 누른다.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_source_4.png)  

그러면 팝업이 하나 더 뜨는데,

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_source_5.png)  

위에 올렸던 [비교 사진](#4-생성한-ik_rig-세부-설정)을 보면,  
우측의 `IK Retargeting` 탭에서 Spine 체인엔 `IK_Goal` 이 없는걸 확인할 수 있음.  
그대로 따라하기 위해 No Goal 을 선택한다.  

### IK_Goal 생성

> 목표(Goal)를 지정해주면, 원본과 타겟 둘 다 같은 곳을 바라본다고 해야할까...
> 아무튼 리타게팅을 할 때 좀 더 잘 변환이 됨.

나머지 항목도 계속 진행하다가,  
Arm 항목의 경우 <b>IK_Goal</b> 이 설정되어 있음.  
`IK_Warrior` 를 눌러보면 Goal 은 손등? 손목 관절?에 설정되어 있음.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_target_hand.png)  

`IK_Canca` 도 똑같이 손등? 부분의 항목을 IK_Goal로 지정한다.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_source_hand.png)  

해당 항목에 우클릭을 하여 지정할 수 있음.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_goal.png)  

선택하면 다른 창이 또 뜨는데...  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_goal_fullbody.png)  

이 부분은 나도 잘 모르겠음.  
일단 기존 IK_Warrior 엔 Fullbody로 되어 있어서, 그대로 OK 했음.  

### Assign IK_Goal

그리고나서 IK_Warrior처럼 IK_Cancan의 Arm 항목들을 SHIFT키로 잡고 Arm 으로 체인 추가를 하면,  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_goal_setting.png)  

체인으로 추가할 항목에 Goal 도 포함되어 있어서 Arm 의 목표를 추가하겠냐는 팝업이 뜬다.  
Assign 을 눌러준다.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_goal_setting_done.png)  

IK_Goal 이 잘 설정되었다.  

## 4-3. 체인 생성 완료

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_rig_final.png)  

IK_Warrior 와 얼추 비슷하게 체인 생성이 끝났다.  
저장하고 창 닫기.  



# 5. IK_Retargeter 생성 및 설정

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_retargeter_create.png)  

다음 IK_Retargeter 를 생성한다.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_retargeter_select.png)  

위에서 만든 IK_Cancan 을 선택한다.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_rtg_cancan.png)  

이름을 RTG_Cancan 으로 변경해주었다.  
실행한다.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_targeter_target.png)  

타겟 에셋을 IK_Warrior로 지정해주면 프리뷰 창에 워리어도 나타나게 된다.  

하단의 `Chain Mapping` 탭을 보면,  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_targeter_Chain_Mapping.png)  

위에서 지정했던 체인 값들이 매칭되는 것을 볼 수 있다.  
다만 Source(Cancan) 에서 root 는 지정하지 않았었기 때문에 None 으로 바꾼다.  
타겟에 대해 소스에 없는 것들은 다 None 으로 바꿈... 

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_ik_targeter_Chain_Mapping_2.png)  

그리고 나서 `Chain Mapping` 탭 옆의 `Asset Browser` 를 눌러서  
cancan 모션을 더블클릭 해보면,  

<img src="https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/CancanWarrior.gif">  

재생이 되는 걸 확인해볼 수 있다~  
저장해주고 창을 끈다.  



# 6. 애니메이션 에셋 리타게팅

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_retarget_ani_select.png)  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_retarget_select.png)  

맨처음 임포트 했었던 애니메이션 에셋을 찾아 우클릭한다.  
언리얼은 Animation Sequence 가 애니메이션 에셋이랜다.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_IK_Retargeter.png)  

그러면 창이 뜨는데, 방금전 만들었던 RTG_Cancan 을 선택한다.  

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_IK_Retargeter_done.png)  

내 캐릭터에 맞춰진 cancan 애니메이션이 생성되었다.  



# 결과

<img src="https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/Mixamo/maximo_result.gif">  

원한대로 적npc는 idle 상태시 캉캉춤을 추고 있게 되었다.  


* 참고 링크
  * [언리얼엔진5 리타게팅 설정 방법](https://www.youtube.com/watch?v=vVGEJdaj2os)
  * [언리얼엔진4_3인칭 캐릭터 리타겟팅_1. 믹사모 캐릭터를 언리얼에 임포트하기](https://www.youtube.com/watch?v=iVPnjoj5nf4)
  * [메타휴먼에 믹사모 애니메이션 적용-1](https://blog.naver.com/john_creator/222491495754)
  * [IK 릭 리타기팅 (공식문서)](https://docs.unrealengine.com/5.0/ko/ik-rig-animation-retargeting-in-unreal-engine/)