---
title:  "[정리] Singleton"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, Singleton]

toc: true
toc_sticky: true
 
date: 2024-02-19
last_modified_at: 2024-02-19
imagefolder: "UnrealDocs/GameSingleton/"
---

<br>

# 언리얼 제공 Singleton 클래스

언리얼 엔진에서 제공해주는 싱글톤 클래스는 다음이 있다.

- GameInstance
  - 게임 컨텐츠보단 어플리케이션의 데이터를 관리하는 용도로 사용함
  - 사용자가 게임을 시작하면 엔진을 초기화하고 가장 먼저 실행하는 오브젝트가 GameInstance입니다.  
  그리고 게임이 종료될 때 까지 GameInstance는 살아있고 프로그램이 종료될 때 가장 마지막에 소멸됩니다.  
  이러한 특징으로 인해 GameInstance의 멤버를 확장해나가면 게임의 전체 라이프싸이클(LifeCycle)에서 사용되는 데이터를 관리할 수 있습니다.  
  GameInstance 오브젝트가 초기화될 때, Init이라는 함수를 호출하는데, 이를 상속받으면 우리가 어플리케이션의 초기화 루틴을 만들 수 있습니다.  
- AssetManager
- GameMode
- GameState

전체 게임 내에서 단 하나만 존재하는 싱글톤 클래스로서,  
어디서든 해당 객체에서 값을 가져올 수 있도록 한다.  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealSingleton1.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealSingleton1.png)  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealSingleton2.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealSingleton2.png)  

Project Setting 에서 등록 가능 하다.  

## GameSingleton

작업 하다가 싱글톤 객체를 사용하려고 클래스 생성 -> `GameInstance` 를 상속 받아서 클래스를 만들어도 되지만,  
위에 써놨듯이 `GameInstance` 는 어플리케이션 데이터를 관리하는 용도로 쓰이기 때문에  
`UObject` 클래스를 상속받아 생성하여 만든 다음 Project Setting에서 singleton class로 등록해주면 된다.  

```cpp
// ABGameSingleton.h ------------------
UCLASS()
class UNREALARENABATTLE_API UABGameSingleton : public UObject
{
	GENERATED_BODY()
	
	
public:
	UABGameSingleton();

	// 모든 클래스에서 이 인스턴스를 가져올 수 있도록 static으로 생성
	static UABGameSingleton& Get();
};


// ABGameSingleton.cpp ------------------
UABGameSingleton& UABGameSingleton::Get()
{
	// 싱글톤 그대로 반환
	UABGameSingleton* Singleton = CastChecked<UABGameSingleton>(GEngine->GameSingleton);
  // Project Setting 에서 등록해놨으므로 GEngine->GameSingleton 으로 접근 가능

	if (Singleton)
	{
		return *Singleton;
	}
	
	UE_LOG(LogABGameSingleton, Error, TEXT("Invalid Game SIngleton"));

	return *NewObject<UABGameSingleton>();
	// 그냥 함수 흐름을 위해 리턴할 뿐, 사용할 순 없음. 그냥 단순 생성만 한거니까;;
}
```

`UObject` 를 상속 받은 클래스를 생성한 후 위의 캡쳐대로 Game Singleton Class 에 등록 해주면 된다.  
이후 코드 내에선 `GEngine->GameSingleton` 코드로 접근할 수 있다.  


### 엔진 내부 코드에서 GameSingleton 등록하는 부분

```cpp
// GEngine->GameSingleton 에서 GEngine 들어가보면, Engine.h 이 열림
// 여기서 virtual void InitializeObjectReferences(); 부분 정의 들어가면 볼 수 있음.

// UnrealEngine.cpp ------------------
// UEngine::InitializeObjectReferences()

if (GameSingleton == nullptr && GameSingletonClassName.ToString().Len() > 0)
	{
		UClass *SingletonClass = LoadClass<UObject>(nullptr, *GameSingletonClassName.ToString());

		if (SingletonClass)
		{
			GameSingleton = NewObject<UObject>(this, SingletonClass);
		}
		else
		{
			UE_LOG(LogEngine, Error, TEXT("Engine config value GameSingletonClassName '%s' is not a valid class name."), *GameSingletonClassName.ToString());
		}
	}
```