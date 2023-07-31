---
title:  "[Unity] InitState (Random.seed 는 더이상 사용되지 않음)"
categories:
  - UnityDocs
tags:
  - [Unity, Game Engine]

toc: false
toc_sticky: false
date: 2023-05-22
last_modified_at: 2023-05-23
---

<br>

`Random` 을 사용하여 난수를 생성할 때, 시드를 사용하면 동일한 패턴의 난수를 생성하게 된다.  

(💡 시드란? 컴퓨터가 난수 생성 알고리즘을 사용하여 난수를 만들 때, 알고리즘에 사용하는 값)  

예전에 이 시드를 생성할 땐 이런식으로 작성했던 것 같다.  


```c#
UnityEngine.Random.seed = System.DateTime.Now.Millisecond;
```

그런데 최근 시드를 생성해야 했어서 찾아보니 이제 저런 식의 코드는 더이상 사용되지 않는 듯 하다.  

```c#
UnityEngine.Random.InitState((int)DateTime.Now.Ticks);
```

InitState를 사용하여 시드를 초기화 해주나보다!