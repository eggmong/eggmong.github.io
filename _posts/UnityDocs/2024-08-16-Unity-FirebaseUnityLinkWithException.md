---
title:  "[Firebase] 파이어베이스 업데이트 후 LinkWith~ 함수 익셉션 변경"
expert: ""

categories:
  - UnityDocs
tags:
  - [Unity, Firebase]

toc: true
toc_sticky: true
 
date: 2024-08-16
last_modified_at: 2024-08-16
---

<br>

1. 파이어베이스를 업데이트 했음 (10.7.0 에서 11.9.0으로)  

2. 그런데 게스트 계정과 구글 계정 연동 이슈가 생김  
2-1. LinkWith 어쩌고가 게스트 계정을 구글 계정에 연동할 때 사용하는 함수임.  

3. 게스트 계정으로 플레이 하다가 구글 계정에 연동하려 할 때, 이 구글 계정이 이미 게임을 플레이 한 적이 있으면 파베에서 익셉션으로 에러를 줌.  

4. 파베가 익셉션으로 에러를 주면, 이 때 게임에선 팝업으로 '이미 구글 계정으로 연동된 계정'이라고 안내를 해주고 있었음.  

5. 그런데 이번에 파베가 업데이트가 되면서 익셉션 주는 게 다른 익셉션으로 변경되버려서 익셉션을 제대로 못잡음.  

6. 그래서 유저들한텐 '연동이 실패했다' 라는 일반적인 팝업이 나와버려서 유저들이 연동 안된다고 CS를 넣음.  

7. 처음엔 파베에 문제가 있는 줄 알고 찾아봤는데, 익셉션이 다른 걸로 바뀐 것이었다- 그래서 바뀐 익셉션을 찾아서 다시 제대로 팝업을 띄워 주었다. 끝.  



추가) 애플 문서엔 제대로 적혀있었다.  
문서가 파편화 되어 있군.  

[https://firebase.google.com/docs/auth/unity/apple?hl=ko](https://firebase.google.com/docs/auth/unity/apple?hl=ko)  

![Firebase]({{ site.imageurl }}UnityDocs/UnityFirebaseLinkWith.png)  