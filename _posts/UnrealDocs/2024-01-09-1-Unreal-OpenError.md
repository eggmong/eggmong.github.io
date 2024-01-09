---
title:  "[Error] 언리얼 프로젝트 .uproject 파일로 열 때 안열림 문제"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine, Error]

toc: true
toc_sticky: true
 
date: 2024-01-08
last_modified_at: 2024-01-08
---

<br>


git에 언리얼 프로젝트를 커밋하고,  
다른 컴에서 열어봤는데 열리지 않았다. (왜?! 아직도 원인은 못찾았다.)  

# 해결 방법

![Error1](https://drive.google.com/uc?export=view&id=1SfAR3ISKxNMin2BLEuWDS_lY0s98rfi_)  

`.uproject` 파일에 우클릭으로 `generate Visual Studio project files` 을 실행한다.  
그러면 `.sln` 솔루션 파일이 생긴다.  

솔루션 파일 눌러서 비쥬얼 스튜디오 실행 후,  
빌드 한 번 해준다.  
별다른 에러가 없다면 빌드가 성공할 것이고,  
Ctrl+F5 눌러서 실행해보면 프로젝트가 열린다.  

이후부턴 `.uproject` 파일로 실행해도 프로젝트가 잘 열린다.  