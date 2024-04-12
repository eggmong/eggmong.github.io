---
title:  "Apply & Clear, 링크 복제(Duplicate Linked), 링크 해제"
excerpt: "초기화하는 Clear, 변환한 모습을 디폴트로 두고 싶을 때 쓰는 Apply, 연결된 복제 생성"

categories:
  - Blender
tags:
  - [Blender, 블렌더]

toc: true
toc_sticky: true
 
date: 2024-04-12
last_modified_at: 2024-04-12
imagefolder: "Blender/ApplyClear/"
---

<br>

# Clear

변환 했던 것을 초기 모습으로 되돌리는 명령어이다.  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear00.png)]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear00.png)  

예를 들어 위치, 크기, 회전값을 바꾼 큐브를 원래의 상태로 되돌리고 싶다면 각각 클리어를 해주면 되는 것이다.  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear01.png)]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear01.png)  
[![Blender]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear02.png)]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear02.png)  

> 💡 Clear 단축키 : `Alt + G` (이동), `Alt + R` (회전), `Alt + S` (크기)




# Apply

변환한 모습 그대로를 디폴트 값으로 두고 싶다면 Apply를 사용하면 된다.  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear04.png)]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear04.png)  

> 💡 Apply 단축키 : `Ctrl + A`

예를 들어 이렇게 변환한 큐브를 디폴트 값으로 두고 싶다면  
`Location` 을 누르면 된다.  
그러면 오브젝트 프로퍼티의 Location 값도 (0, 0, 0) 으로 바뀐 것을 볼 수 있을 것이다.  

<b>이 말은 즉 오브젝트의 로컬 축이 글로벌 축과 동일해졌음을 뜻한다.  </b>

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear05.png)]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear05.png)  
[![Blender]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear06.png)]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear06.png)  
[![Blender]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear07.png)]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear07.png)  




# 링크 복제 (Duplicate Linked)

단순한 복제는 오브젝트 선택 후 `Shift + D` 를 누르면 되지만,  
연결된 복제는 `Alt + D` 를 누르면 된다.  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear08.png)]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear08.png)  

연결된 복제는 변형을 해보면 기존 것과 똑같이 변형된다.  

또한 재질도 링크가 되어서 같이 바뀌게 된다.  

> ❗단, <mark style='background-color: #fff5b1'>오브젝트 모드에서의 변형은 링크되지 않는다.</mark>  
> 오직 에디트 모드에서의 변형만 링크된다.  
> 그리고 <mark style='background-color: #fff5b1'>모디파이어 적용도 따로 링크된다.</mark>  


## 링크 해제하기

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear09.png)]({{ site.imageurl }}{{ page.imagefolder }}ApplyClear09.png)  

`Object -> Releations -> Make Single User -> Object & Data` 선택하면 링크가 해제된다.  