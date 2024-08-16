---
title:  "[Animation] 커브를 Linear로 바꿔줘야 딱딱 움직이는 애니메이션이 된다"
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

유니티에서 오브젝트에 애니메이션을 생성하면  
애니메이션에 기본적으로 <b>커브(Curve)</b>가 들어가 있다.  

커브가 들어 있으면 애니메이션이 해당 커브 곡선에 따라 반동이 일어나며 움직이게 되는데,  
반동 없이 딱딱 재생되는 애니메이션으로 만들고 싶다면  
해당 키를 잡고 <b>Linear</b> 으로 바꿔줘야 딱딱 움직이는 애니메이션이 만들어진다.  

![Animation]({{ site.imageurl }}UnityDocs/UnityAnimationCurveLinear1.png)  

Linear를 선택하자.  

![Animation]({{ site.imageurl }}UnityDocs/UnityAnimationCurveLinear2.png)  
![Animation]({{ site.imageurl }}UnityDocs/UnityAnimationCurveLinear3.png)  

곡선이 직선이 된다.  