---
title:  "[Error] [Build] UnityException: GetRegistryStringValue can only be called from the main thread. "
categories:
  - UnityDocs
tags:
  - [Unity, Game Engine, Build, Error]

toc: false
toc_sticky: false
date: 2023-08-31
last_modified_at: 2023-08-31
---

<br>


> UnityException: GetRegistryStringValue can only be called from the main thread.  
> Constructors and field initializers will be executed from the loading thread when loading a scene.  
> Don't use this function in the constructor or field initializers, instead move initialization code to the Awake or Start function.  

개발중인 게임 빌드를 뽑아서 테스트 해보려고 했더니 이런 에러가 뜨며 빌드가 되지 않았다.  

처음엔 에러 로그에서 `main thread`라고 하길래, 비동기 함수를 사용하면서 문제가 생겼나 생각하며 삽질 좀 했다.  

그러다  Build Setting 창에서 `Windows, Mac, Linux` 를 선택하면 뜨지 않고  
`Android` 를 선택하면 에러가 뜨길래  
Android 세팅이 잘못되었나보다고 추측할 수 있었다.  
이것 저것 들쑤셔본 결과 이유는 간단했다.  

![JDKError](https://drive.google.com/uc?export=view&id=1HsQWkaq-Aq5-pxVAWD04Qd2pVlUMbqKa)  

캡쳐에선 채워져있지만... JDK 항목이 뜬금없이 비워져 있었다. ;;  
다시 채워주니 잘 작동한다.  