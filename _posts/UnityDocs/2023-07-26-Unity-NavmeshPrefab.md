---
title:  "[Error] [NavMesh] 동적으로 생성한 프리팹에서 네브메쉬 굽기 (Navigation Static) "
categories:
  - UnityDocs
tags:
  - [Unity, Game Engine, NavMesh, Error]

toc: false
toc_sticky: false
date: 2023-07-28
last_modified_at: 2023-07-28
---

<br>

> 💡 내가 작업한 하우징 시스템의 경우 유저마다 배치된 아이템의 정보가 다르기 때문에,  
>    동적으로 네브메쉬를 만들어야 한다.  
>    동적으로 네브메쉬 만드는 법을 찾아가며 해보다가 발견한 이슈!

_어찌된 영문인지 모르겠지만_ fbx 파일 그대로 바닥에 올려놓고 NavMesh를 만들면 메쉬 정보를 읽어들이는데,  

fbx파일을 프리팹으로 만든 상태에서 바닥에 올려놓으면 NavMesh가 프리팹의 메쉬 정보를 못읽어들인다.  

그래서 NavMesh를 만들면 이렇게 되버린다.  

![NavMesh]({{ site.imageurl }}UnityDocs/2023-07-26_NavMesh1.png)  

왼쪽이 FBX 파일 그대로 올린 것, 오른쪽이 프리팹 상태로 올린 것이다.  

네브메쉬가 제대로 생성되지 않았다.  

<br>

해결 방법은, 먼저 유니티 상단메뉴의 Window -> AI -> Navigation 을 눌러 창을 띄운 후,  
프리팹을 선택하고 이걸 체크해준다.  

![NavMesh]({{ site.imageurl }}UnityDocs/2023-07-26_NavMesh2.png)  

그러면 메쉬 정보를 잘 읽어들인다!