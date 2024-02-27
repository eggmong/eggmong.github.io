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
GAS 프레임워크는 PlayerState에서 ASC를 생성하고, 이 생성된 ASC 포인터를 캐릭터에 넘겨주는 것이 바람직하다.  

발동한 캐릭터의 어빌리티들은 대부분 모두 캐릭터 중심으로 진행될 것이고  
어빌리티로 인한 결과는 ASC가 데이터로 보관하고, 이것들은 실시간으로 서버에서 클라이언트로 전송될 것이다.  

***

## GameMode 세팅

[![GA]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase2.png)]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase2.png)  

GameModeBase를 상속받은 블루프린트 클래스를 만든 뒤,  
PlayerController 와 PlayerState 넣어주고, 플레이어 캐릭터도 넣어준다.  

[![GA]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase3.png)]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase3.png)  

월드 세팅에서 GameMode 넣어준다.  

[![GA]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase4.png)]({{ site.imageurl }}{{ page.imagefolder }}GASCharacterBase4.png)  

프로젝트 세팅에서도 GameMode 넣어준다.  

