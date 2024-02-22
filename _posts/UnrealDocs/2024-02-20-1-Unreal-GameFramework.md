---
title:  "[정리] 언리얼 Game Framework"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: false
toc_sticky: false
 
date: 2024-02-20
last_modified_at: 2024-02-20
imagefolder: "UnrealDocs/GameFramework/"
---

<br>

엔진의 시작점은 Launch.cpp의 `GuardedMain` 함수이다.  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework2.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework2.png)  

cpp 파일의 여러 코드들을 걷어내고 눈여겨볼 부분만 남겨놓으면,  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework1.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework1.png)  

이렇게 볼 수 있다.  

엔진의 메인 루프는 `FEngineLoop` 라는 클래스에서 구현된다.

엔진 루프에 `PreInit` 단계가 있고,  
엔진이 완전히 초기화 된 다음 종료할 준비가 될 때까지 모든 프레임을 체크(Tick) 하는 것을 볼 수 있다.  

`PreInit` 함수는 대부분의 모듈이 로드되는 곳 이다.

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework3.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework3.png)  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework3_1.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework3_1.png)  

***

C++ 소스 코드가 포함된 게임 프로젝트나 플러그인을 만들때 소스 모듈을 정의하면 `LoadingPhase` 를 지정하여 해당 모듈이 로드되는 시점을 정할 수 있다.

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework4.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework4.png)  

`LoadingPhase` 는 ModuleDescriptor 에서 지정해놓았다.

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework5.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework5.png)  


엔진은 다양한 소스 모듈로 분할된다.  

일부 모듈은 다른 모듈보다 더 일찍 초기화되어야 하며, 일부는 특정 플랫폼이나 특정 상황에서만 로드된다.  

따라서 모듈 시스템은 코드베이스에서 여러 부분간의 종속성을 관리할 수 있도록하고  
특정 구성에 필요한 것만 로드할 수 있도록 해준다.  

***

`FEngineLoop` 가 `PreInit` 단계를 시작하면  
일부 하위 수준 엔진 모듈을 로드하여 필수 시스템이 초기화 되고 필수 유형이 정의된다.  

일단 첫번째로 `LoadCoreModules` 를 통해 CoreUObject가 로드된다.  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework6.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework6.png)  


그리고 `LoadPreInitModules` 함수로 필수 모듈들을 로드한다.

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework7.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework7.png)  


그런 다음 프로젝트 또는 활성화된 플러그인에 '초기 로드' 순서의 모듈이 있는 경우, 해당 모듈이 다음에 로드된다.  
(`LoadStartupCoreModules`)

그다음 더 높은 수준의 엔진 모듈들이 로드된다.  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework8.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework8.png)  

***

언리얼 엔진의 `PreInit` 에서 대부분의 모듈을 로드한다.  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework9.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework9.png)  

게임 모듈은 모든 필수 엔진 기능이 로드되고 초기화되는 시점에 생성되지만  
실제 게임 상태가 생성되기 전에 이루어진다.  
  
모듈이 로드되고 난 후,  
  
먼저 엔진은 해당 모듈에 정의된 UObject 클래스들을 등록한다.  

이렇게 하면 리플렉션 시스템이 해당 클래스를 인식하여 각 클래스에 대해 Class Default Object를 구성한다.  

<b>CDO(Class Default Object)</b> 는 기본 상태의 클래스 개체이며, 추가 상속을 위한 프로토타입 역할을 한다.  

따라서 사용자 정의 Actor나 GameMode, 또는 `UCLASS` 키워드를 사용하여 선언한 모든 것을 정의했다면  
엔진 루프는 해당 클래스의 CDO를 할당한 다음에 생성자를 실행하여 상위 클래스의 CDO를 템플릿으로 전달한다.  

게임 플레이 관련 코드가 생성자에 포함되면, 상위 클래스나 다른 멤버 변수들이 초기화되지 않은 상태에서 해당 코드가 실행될 수 있다.  

그렇기 때문에 생성자에 게임 플레이 관련 코드가 포함되서는 안된다.  

***

`FEngineLoop`의 `PreInit` 을 통해  
필요한 모든 엔진, 프로젝트 및 플러그인 모듈을 로드하고 클래스들을 등록하고 하위 수준 시스템들을 초기화 했다면  
다음 단계인 `Init()` 으로 넘어갈 수 있다.  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework10.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework10.png)  

Init을 단순화하면 이렇게 나타낼 수 있다.  

`GEngine = NewObject<UEngine>(GetTransientPackage(), EngineClass);`
이 구문을 통해 `UEngine` 이라는 클래스에 작업을 전달하는 것을 볼 수 있다.  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework11.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework11.png)  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework11_1.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework11_1.png)  

[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework11_2.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework11_2.png)  

`UEngine` 클래스는 `UEditorEngine` 과 `UGameEngine` 에서 상속받아 구현되는 클래스이다.  



[![bt]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework12.png)]({{ site.imageurl }}{{ page.imagefolder }}UnrealGameFramework12.png)  

처음 FEngineLoop::Init() 에 진입하면, 엔진 구성 파일을 확인하여 어떤 Engine 클래스를 사용해야 하는지 파악한다.  
그런 다음 해당 클래스의 인스턴스를 생성하고, 이를 전역변수 GEngine을 통해 액세스 할 수 있는  
전역 UEngine 인스턴스로 보호한다.  





***

참고: [https://youtu.be/IaU2Hue-ApI?si=qKTo-oVoIDgDeIOa](https://youtu.be/IaU2Hue-ApI?si=qKTo-oVoIDgDeIOa)