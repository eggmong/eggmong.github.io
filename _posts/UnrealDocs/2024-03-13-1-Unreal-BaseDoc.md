---
title:  "[정리] 언리얼 엔진 기초 개념 정리 (World, Actor, Pawn, Input, GameMode, Character)"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, World, Actor, Pawn, Input, GameMode, Character]

toc: true
toc_sticky: true
 
date: 2024-03-13
last_modified_at: 2024-03-13
---

<br>


UObjectBase 클래스가 최상위 클래스.  

이걸 UObjectBaseUtility가 상속 받음.  

UObjectBaseUtility를 또 UObject가 상속 받고,  

UObject 클래스를 AActor 클래스가 상속 받음.  

class UStaticMeshComponent 라는 클래스의 상속 구조를 보면,  
UActorComponent를 상속 받는데  
얘도 결국 UObject 클래스를 상속 받음..  

> UObject 클래스를 상속받으면, 언리얼의 가비지컬렉션에 의해 스마트 포인터로 메모리 관리를 할 수 있음.  


# 월드 (World)

- 게임 콘텐츠를 담기 위해 제공되는 가상의 공간
- 월드는 시간, 트랜스폼, 틱을 서비스로 제공한다.
- 월드 세팅이라는 콘텐츠 제작을 위한 기본 환경 설정을 제공한다.
- 월드의 기본 단위는 <u>액터(Actor)</u>>로 정의되며, 액터 클래스는 접두사 `A` 를 사용한다.
- Transform, Tick, Time, World Setting, Actor



# 게임 모드 (GameMode)

- 게임 규칙을 지정하고 게임을 판정하는 최고 관리자 액터.
- 형태가 없음! 논리적인 단위로만 존재.
- 언리얼 엔진에서 <b>하나의 게임에는 반드시 하나의 게임 모드만 존재</b>한다.
- 게임 모드에서 입장할 사용자의 규격을 지정할 수 있다.
- 멀티플레이 게임에서 판정을 처리하는 심판 역할
- 게임모드엔 UI 생성하는 함수 같은 생성 금지.  
  데디 같은 서버가 붙게 되면, 게임모드는 클라이언트에 생성이 안 됨. 서버에만 생성됨.
- 내부 클래스 종류  
  - Game Session Class
        유저랑 서버랑 연결되있는 통신 도구. 서버와 유저가 세션을 통해 데이터를 주고받을 수 있음.  
        로그인 등 처리.  
  - Game State Class (Actor)  
      게임의 상태를 다루기 위한 클래스. 자동으로 스폰됨.  
      게임모드와 마찬가지로 서버에만 만들어짐.  
  - PlayerController  
      폰에 빙의해서 플레이어가 조작할 수 있게 만들어주는 클래스.  
  - PlayerStateClass  
      마찬가지로 Actor로 만들어져 있음. 플레이어 정보를 저장하기 위한 클래스.  
      플레이어의 공격력, 방어력 같은 것을 저장하고,
      이걸 끌어다가 사용할 수 있음.  
  - HUD class
      hud ui 클래스. 근데 보통 ui는 코드로 관리할거라, 이거 잘 안 쓴다고 함.  
  - Default Pawn Class
      기본으로 어떤 폰을 사용할거냐- 는 클래스. 플레이어가 빙의할 기본 폰 클래스를 지정할 수 있음.  
      기본으로 DefaultPawn 이 지정되어 있어서, 디폴트 씬 실행해보면 이게 생성 됨.  
  - Spectator class
      플레이어가 관전자 상태일 때 사용하는 폰 클래스.  
  - Re머시기 클래스. 서버 쪽 통계 관리 클래스.  


# 폰 (Pawn)

- 무형의 액터인 플레이어가 빙의해 조종하는 액터 <b>(`AActor`를 상속 받음)</b>
- 길찾기를 사용할 수 있으며, 기믹 및 다른 폰과 상호작용한다.
- 폰 중에서 인간형 폰을 별도로 캐릭터(ACharacter) 라고 지칭한다.
- <b>빙의</b>를 통해 플레이어와 연결, 사용자의 입력 처리, 화면에 대응하는 카메라 설정, 기믹과 상호작용, 애니 재생
  - PlayerController, AIController 2가지 컨트롤러로 폰을 조종할 수 있음.  
  - 디폴트 씬에서 그냥 플레이해보면, 기본 생성된 PlayerController가 기본 생성된 DefaultPawn에 빙의되어 조종할 수 있음.

## 캐릭터 (ACharacter)

인간형 폰을 구성하도록 언리얼이 제공하는 전문 폰 클래스이다.  
- 기믹과 상호작용을 담당하는 캡슐 컴포넌트 (루트 컴포넌트)
- 애니메이션 캐릭터를 표현하는 스켈레탈 메시 컴포넌트
- 캐릭터의 움직임을 담당하는 캐릭터 무브먼트 컴포넌트
- `ACharacter` 클래스는 플레이어로 구현하기 편하도록 여러 기능이 포함되어 있지만,  
  그만큼 쓰지 않을 수도 있는 기능들도 다 들어있으므로,  
  많이 생성하면 프레임이 떨어질 수 있음. 그래서 보통 플레이어 자신만 이 클래스로 생성함.  
  - 캐릭터 클래스 전용 컴포넌트: Character Movement. 이거 폰 같은 데선 못 씀.  


# 액터 (Actor) 의 구조

- <u>월드의 속한 콘텐츠의 기본 단위</u>를 액터라고 한다.
- 액터는 트랜스폼을 가지며, 월드로부터 틱과 시간 서비스를 제공 받는다.
- 액터는 다수의 컴포넌트를 가지고 있으며, 컴포넌트롤 통해 구현을 하고, 액터는 사실 컴포넌트를 감싸고 있는 형태인 것이다.
- 다수의 컴포넌트를 대표하는 컴포넌트를 Root Component라고 한다.
- 액터는 Root Component를 가져야 하며, 루트 컴포넌트의 트랜스폼은 즉 액터의 트랜스폼이 된다.



## C++ 액터에서 컴포넌트의 생성

- 컴포넌트는 언리얼 오브젝트이므로 `UPROPERTY` 를 설정하고 `TObjectPtr` 로 포인터를 선언한다.
- CDO에서 생성된 컴포넌트는 자동으로 월드에 등록된다.
- `NewObject` 로 생성한 컴포넌트는 반드시 등록절차를 거쳐야 한다. (예: RegisterComponent)
- 등록된 컴포넌트는 월드의 기능을 사용할 수 있으며, 물리와 렌더링 처리에 합류한다.
- 컴포넌트 지정자
  - Visible / Edit : 객체타입과 값타입으로 사용
  - Anywhere / DefaultsOnly / InstanceOnly : 에디터에서 편집 가능 영역
  - BlueprintReadOnly / BlueprintReadWrite : 블루프린트로 확장시 읽기 혹은 읽기쓰기 권한을 부여
  - Category : 에디터 편집 영역(Detail) 에서의 카테고리 지정