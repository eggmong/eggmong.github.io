---
title:  "미피 모델링"
excerpt: "https://youtu.be/f1YFQR9TttA?si=5U8cO3zsPPI87ZR0"

categories:
  - Blender
tags:
  - [Blender, 블렌더]

toc: true
toc_sticky: true
 
date: 2024-04-30
last_modified_at: 2024-04-30
imagefolder: "Blender/"
---

<br>

큐브 선택 -> Ctrl + 2  
	Subdivision Modifier 2단계. 큐브가 원형으로 바뀐다. 면을 잘게 나눈다.  
	하지만 메시는 여전히 사각형인 상태.  
	* 모디파이어란, 실제 메시는 바꾸지 않고 겉으로만 보이도록 해주는 것.  
	
	모디파이어 상태를 Apply로 메쉬화 할 수 있다.  
	오브젝트 모드(Tab) 에서, Modifier 탭에서 드롭다운 메뉴에서 Apply 누르면 됨.  
	
Alt + Z : 엑스레이 모드  
이걸로 에딧 모드에서 점들을 선택하여 지울 수 있다.  
그리고 Mirror modifier 를 추가하여 반대와 똑같이 만들어지도록 할 수 있음. 대칭.  
그런데 모디파이어에서 Clipping 을 선택해줘야 반대쪽의 점들이 서로 붙는다!  

GZ : 면을 만두처럼 수정 가능. (Z를 눌러 Z값은 고정한다는 뜻, G를 눌러 이동한다는 뜻)  
	이걸 하려면 Proportional Editing 을 해줘야 하는데 안될 때가 있음.  
	그럴 경우 단축키 O(영어 오) 를 누르면 이 기능이 켜진다...  

SY : 사이즈 줄이기, Y는 고정으로.    

I : inset face. 면 안에 면을 또 만들어주는 기능. 토끼 귀 만들 때 면을 선택 후 이걸 선택하였따..  

E : extrude. 면 뽑기. 면 돌출. 토끼 귀 돌출시킴.  

Ctrl + r : 루프 컷. 루프 선 추가. 선과 선 사이(?)에 선 추가. 이러면 모양이 각지게 잘 잡힘. 토끼 귀 모양 잡을 때 썼음.  

GG : 선택한 선 또는 점 등을 이미 위아래 혹은 양옆에 잡힌 영역 사이에서 왔다갔따 하게 해줌.  

Ctrl + B : 베벨...? 루프컷 추가 후 베벨하면 모양이 유지가 되면서... 추가가 됨...  
			근데 베벨은 보통 선 베벨이고, 점 베벨을 하려면 점 선택 후 Ctrl + B + V 까지. 눌러줘야.  
			
Solidify 모디파이어 : 두께감  


Alt + R : Clear. 에디트 모드에서 했던 로테이션이 초기화 됨.  