---
title:  "[정리] 캐릭터 애니메이션 시스템 설계"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, Animation]

toc: true
toc_sticky: true
 
date: 2024-08-27
last_modified_at: 2024-08-27
---

<br>

UAnimInstance 클래스를 상속 받아서 자신이 사용할 애님 인스턴스 클래스를 생성한 후,  
이 클래스를 바탕으로 블루프린트를 만들어서 캐릭터에서 사용한다.  

- 캐릭터는 GetAnimInstance() 함수를 통해 AnimInstance 를 얻을 수 있음.

```cpp
// 먼저, ACharacter를 상속받은 캐릭터 클래스에서 애니메이션 인스턴스 설정
static ConstructorHelpers::FClassFinder<UAnimInstance> AnimInstanceClassRef(TEXT("/Game/프로젝트이름/Animation/ABP_SP_Character.ABP_SP_Character_C"));
if (AnimInstanceClassRef.Class)
{
	GetMesh()->SetAnimInstanceClass(AnimInstanceClassRef.Class);
}
```

- AnimInstance는 GetOwningActor() 함수를 사용해서 자신을 소유한 액터 정보를 얻을 수 있음.

```cpp
// 자신(애님인스턴스)을 소유한 액터의 정보 가져오기
Owner = Cast<ACharacter>(GetOwningActor());
if (Owner)
{
	Movement = Owner->GetCharacterMovement();
}
```

애니메이션 블루프린트는 <b>이벤트 그래프</b>와 <b>애님 그래프</b>, 두 영역으로 구성되어 있음.  
- 이벤트 그래프 : 이벤트로부터 상태를 파악할 수 있는 주요 변수를 저장.  
    - 주요 이벤트
        - NativeInitializeAnimation : 처음 애니메이션 시스템이 초기화될 때 발생
        - NativeUpdateAnimation : 프레임마다 발생
    
- 애님 그래프 : 위의 이벤트 함수로 인해 저장된 변수를 참조하여, 해당 변수에 지정된 상태의 애니메이션을 재생하도록 우리가 설계해주면 된다.
    <b>State Alias</b>라는 기능을 사용해서 설계해주면 됨. 모드를 분리해서 설계할 수 있다.  
    (땅 위를 걸어다닐 때의 모드, 점프하거나 떨어질 때 등 땅 위에 있지 않을 때의 모드 같이)  

