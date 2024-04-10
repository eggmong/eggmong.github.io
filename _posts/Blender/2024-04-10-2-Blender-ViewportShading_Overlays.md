---
title:  "뷰포트 셰이딩 & 오버레이 사용하기"
excerpt: "오브젝트들을 구분하기 위해 색 넣어주고, 뷰포트에서 숨기는 법"

categories:
  - Blender
tags:
  - [Blender, 블렌더]

toc: true
toc_sticky: true
 
date: 2024-04-10
last_modified_at: 2024-04-10
imagefolder: "Blender/ViewportSetting/"
---

<br>

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Viewport00.png)]({{ site.imageurl }}{{ page.imagefolder }}Viewport00.png)  
[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Viewport00_1.png)]({{ site.imageurl }}{{ page.imagefolder }}Viewport00_1.png)  

각 오브젝트들을 구분하기 위해 랜덤으로 색을 넣는다거나  
조각을 할 때 그림자를 넣어 입체감을 넣고  
어떤 건 보이게 하고 어떤 건 보이지 않게 하도록 하는 방법이다.  


# Viewport Shading


[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Viewport01.png)]({{ site.imageurl }}{{ page.imagefolder }}Viewport01.png)  

현재 솔리드 상태로 렌더되고 있는 상태인 것이다.  
'워크벤치' 라는 렌더 엔진으로 렌더가 되고 있는건데,  
옆에 토글 메뉴를 눌러보면 여러가지 옵션이 뜬다.  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Viewport02.png)]({{ site.imageurl }}{{ page.imagefolder }}Viewport02.png)  

실제 렌더되는 것과는 전혀 상관 없고, 모델링 할 때 편하게 보기 위한 옵션이다.  
작업할 때 편하게 보이는 옵션을 선택하여 볼 수 있다.  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Viewport03.png)]({{ site.imageurl }}{{ page.imagefolder }}Viewport03.png)  

모델링의 오브젝트마다 색을 랜덤으로 넣어줄 수도 있다.  (위 캡쳐)




# Viewport Overlay

뷰포트에서 보이게 될 항목들을 체크할 수 있다.  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Viewport04.png)]({{ site.imageurl }}{{ page.imagefolder }}Viewport04.png)  

`Statistics` 옵션 : 오브젝트와 지오메트리에 대한 정보를 출력할 수 있다.  
(점, 선, 면의 갯수 등)

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Viewport05.png)]({{ site.imageurl }}{{ page.imagefolder }}Viewport05.png)  

`Relationship Lines` 옵션 : Parent 나 Constraint처럼 연결된 관계를 점선으로 표현해준다.  

`Face Orientation` 옵션 : 앞면(파랑)과 뒷면(빨강)을 표시하며, 모든 물체가 파란색인 것이 전형적이다.  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Viewport06.png)]({{ site.imageurl }}{{ page.imagefolder }}Viewport06.png)  