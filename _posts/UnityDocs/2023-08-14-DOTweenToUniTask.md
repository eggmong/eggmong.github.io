---
title:  "[Error] [UniTask] [DOTween] DOTween 을 UniTask 로 변환 "
categories:
  - UnityDocs
tags:
  - [Unity, Game Engine, UniTask, DOTween, Error]

toc: false
toc_sticky: false
date: 2023-08-14
last_modified_at: 2023-08-14
---

<br>

요즘 1인 개발 게임을 만들고 있다.  
프로젝트에 UniTask와 DOTWeen도 사용하고 있는데,  
DOTWeen을 ToUniTask 로 변환할 수 있대서... 해봤는데 안되는 것이다 ㅡㅡ  

알고보니
Editor의 Scripting Define Symbols에  
UNITASK_DOTWEEN_SUPPORT를 추가해야 했다. 그래야 확장함수가 뜬다.  

![ToUniTask](https://drive.google.com/uc?export=view&id=18SVlhB-ytyVWPgtG2Yuepwqf1REz_p7P)

```c#
await tweener
    .OnKill(() =>
    {
        Debug.Log("Fade was OnKill.");
    })
    .OnComplete(() =>
    {
        Debug.Log("Fade was OnComplete.");
    }).ToUniTask();
```

잘 된다.