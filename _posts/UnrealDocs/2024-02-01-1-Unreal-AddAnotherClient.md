---
title:  "ListenServer 사용할 때 Add another client 메뉴 띄우기"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, Listen Server, PIE]

toc: true
toc_sticky: true
 
date: 2024-02-01
last_modified_at: 2024-02-01
---

<br>

![bt]({{ site.imageurl }}UnrealDocs/listenserver_addclient_0.png)  

멀티플레이 게임 구현을 위해 리슨서버를 사용하려면  
옵션에서 저걸 선택하면 됨.  

![bt]({{ site.imageurl }}UnrealDocs/listenserver_addclient_1.png)  

그다음 뷰포트로 실행하고 난 뒤 저 `Add another client` 버튼을 누르면  
새로운 클라이언트가 해당 레벨에 접속하여 추가가 되는데,  

<span style="font-size: 40px"> 내 에디터에선 눈 씻고 찾아봐도 안보였음!!!ㅡㅡ</span>

![bt]({{ site.imageurl }}UnrealDocs/listenserver_addclient_2.png)  

Advanced Setting 들어가서,  

![bt]({{ site.imageurl }}UnrealDocs/listenserver_addclient_3.png)  

`Allow late joining`

이거 체크해주면 됨. ㅡㅡ


* 참고 링크
  * [https://forums.unrealengine.com/t/apparently-there-is-a-button-for-adding-clients/683432/2](https://forums.unrealengine.com/t/apparently-there-is-a-button-for-adding-clients/683432/2)