---
title:  "블렌더로 fbx에 root 추가하기"

categories:
  - UnrealDocs
tags:
  - [blender, fbx, root]

toc: true
toc_sticky: true
 
date: 2024-02-14
last_modified_at: 2024-02-14
imagefolder: "BlenderRootEdit/"
---

<br>

가지고 있던 모델 fbx를 언리얼에 임포트해보니  
root 포지션이 가랑이 사이가 아니라 다른 쪽에 있었다.  

그래서 깔끔하게 지워주고 새로 만들기로 함.  

![bt]({{ site.imageurl }}UnrealDocs/{{ page.imagefolder }}BlenderEditMode1.png)  

블렌더 켜서 Edit Mode로 전환한 다음  
(루트가 bone 으로 잡혀 있어서 Edit Mode로 전환해야 지울 수 있음)

![bt]({{ site.imageurl }}UnrealDocs/{{ page.imagefolder }}BlenderEditMode2.png)  

root 에 해당하는 본에 오른쪽 마우스 클릭 (뷰포트? 에서 해야 함)

![bt]({{ site.imageurl }}UnrealDocs/{{ page.imagefolder }}BlenderEditMode3.png)  

삭제!



![bt]({{ site.imageurl }}UnrealDocs/{{ page.imagefolder }}BlenderEditMode4.png)  

다시 Object Mode로 전환한 다음

![bt]({{ site.imageurl }}UnrealDocs/{{ page.imagefolder }}BlenderEditMode5.png)  

상단에서 Add - Empty - 저거 선택 ㅋ

![bt]({{ site.imageurl }}UnrealDocs/{{ page.imagefolder }}BlenderEditMode6.png)  

생성한 Empty에 Armarture 옮기기.  
말풍선에 써있듯이 `Shift` 키를 누른채 마우스를 떼야 Empty가 부모로 설정 됨.

![bt]({{ site.imageurl }}UnrealDocs/{{ page.imagefolder }}BlenderEditMode7.png)  

이름 바꿔줬음. Root 로.

![bt]({{ site.imageurl }}UnrealDocs/{{ page.imagefolder }}BlenderEditMode8.png)  

익스포트 옵션! 이거 중요함.  
-Z Forward, Up Y 하고~  
Add Leaf Bone 꺼주고~  
Export 하면 됨.  