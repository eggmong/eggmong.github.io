---
title:  "오리진"
excerpt: "면 선택 단축키 : 3, 복사 단축키 : Shift+D, Z기준 단축키 : Z, 반복 단축키 : Shift+R, Add 단축키 : Shift+A"

categories:
  - Blender
tags:
  - [Blender, 블렌더, 단축키]

toc: true
toc_sticky: true
 
date: 2024-04-11
last_modified_at: 2024-04-11
imagefolder: "Blender/Origin/"
---

<br>

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin01.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin01.png)  

오브젝트 가운데에 있는 노란색 점을 오리진이라고 한다.  
오리진을 기준으로 위치가 변하고, 크기가 변하고, 회전이 변하는 것이다.  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin02.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin02.png)  

Origin은 <mark style='background-color: #fff5b1'>오브젝트 모드</mark>와 <mark style='background-color: #fff5b1'>에디트 모드</mark>에서 '변형'할 때 차이가 있다.  

> 변형(Transform) = 옮기고(Move), 회전하고(Rotate), 크기를 바꾸는 것(Scale)  
> 👉 오브젝트 모드 : 오리진도 함께 '변형'  
> 👉 에디트 모드 : 오리진은 '변형'하지 않음  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin03.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin03.png)  

Object Properties에 있는 Transform 수치는 <mark style='background-color: #fff5b1'>Origin</mark>에 대한 수치이다.

그래서 `G` (이동), `R` (회전), `S` (크기) 로 오브젝트를 수정해보면 오리진도 같이 변화한다.  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin04.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin04.png)  

하지만 <mark style='background-color: #fff5b1'>에디트 모드</mark> 에선 오리진이 같이 변하지 않으므로,  
G, R, S로 오브젝트를 수정해도 프로퍼티에 있는 Transform 수치는 변하지 않는다.


에디트 모드에서 오브젝트를 오리진에서 벗어난 위치로 이동시켰다가  
해당 위치로 오리진을 맞춰주고 싶다면,  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin05.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin05.png)  

오브젝트에서 `우클릭 -> Set Origin -> Origin to Geometry` 를 해주면 오리진이 오브젝트 위치로 옮겨진다.  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin06.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin06.png)  



# 단축키를 사용한 오브젝트 수정 작업

> 오브젝트의 면 선택 : Edit 모드에서 단축키 `3`  
> [![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin07.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin07.png)  

> 오브젝트 복제 : `Shift + D`  
> [![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin08.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin08.png)  

> 오리진을 <b>Z 기준</b>으로 <b>회전</b> : `R` (rotate) , `Z` (Z 기준), `45` (45도)  
> [![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin09.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin09.png)  

> 마지막에 했던 행동 반복 (Repeat Last) : `Shift + R`  
> [![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin10.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin10.png)  



Edit 모드(`Tab`누르면 바뀜!) 에서 `1` 을 눌러 점 선택으로 바꾸고,  
한 점을 누른 뒤 `G` 를 눌러 이동해보면 점 하나가 늘어난다.  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin11.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin11.png)  

이 때 시메트리(Symmetry, 대칭) 에서 X 를 켜주면,  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin12.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin12.png)  

<mark style='background-color: #fff5b1'>오리진을 기준으로 반대편 X축에 있는 점이 함께 대칭되어 움직인다!</mark>

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin13.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin13.png)  



# 베벨 모디파이어를 통해 오리진의 차이 보기

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin15.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin15.png)  

큐브에 베벨 모디파이어를 추가하여 오리진의 차이에 대해 알아보자.  
베벨 모디파이어는 오리진에 영향을 받는 모디파이어이다.  

<mark style='background-color: #fff5b1'>오브젝트 모드에선 오리진이 같이 변하므로,</mark>  
오브젝트 모드에서 스케일을 2배로 하면 베벨도 영향을 받아 같이 커진다.  

이 상황에서 에디트 모드로 바꾼 뒤 스케일을 0.5로 바꾸면  
큐브 스케일은 다시 원래 크기로 돌아오지만,  
<mark style='background-color: #fff5b1'>에디트 모드는 오리진은 변하지 않기 때문에</mark> 베벨은 여전히 2배로 커진 상태가 된다.  

[![Blender]({{ site.imageurl }}{{ page.imagefolder }}Origin14.png)]({{ site.imageurl }}{{ page.imagefolder }}Origin14.png)  

그래서 결과적으로 이런 모양이 나오게 된다.  