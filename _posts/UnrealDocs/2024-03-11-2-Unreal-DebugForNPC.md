---
title:  "플레이어가 아닌 npc의 정보 Debug로 확인하기 (DefaultGame.ini)"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, DefaultGame, Debug]

toc: true
toc_sticky: true
 
date: 2024-03-11
last_modified_at: 2024-03-11
---

<br>

DefaultGame.ini 파일을 열어서

```cpp
[/Script/GameplayAbilities.AbilitySystemGlobals]
bUseDebugTargetFromHud=True
```

하단에 위의 코드를 작성해준다.

GameplayAbilities 라고 하는 모듈에 AbilitySystemGlobals 라고 하는 전역적인 설정을 해주는 객체가 있다.  
여기에서 bUseDebugTargetFromHud 를 true로 해주면  
Player가 아닌 다른 액터의 상태 태그를 볼 수 있다.  

[![GA]({{ site.imageurl }}UnrealDocs/bUseDebugTargetFromHud.png)]({{ site.imageurl }}UnrealDocs/bUseDebugTargetFromHud.png)  

npc의 태그가 표시될 것이다. (지금은 상태 설정이 안되어있어서 안떴다)  