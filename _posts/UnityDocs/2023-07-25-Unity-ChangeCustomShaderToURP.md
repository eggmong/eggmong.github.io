---
title:  "[URP Shader] Custom Shader 를 URP Shader로 변경하기까지 과정"
categories:
  - UnityDocs
tags:
  - [Unity, Game Engine, Shader, URP]

toc: false
toc_sticky: false
date: 2023-07-25
last_modified_at: 2023-07-25
---

<br>

이제 유니티 프로젝트에서 셰이더는 URP 파이프라인을 따르는 게 기본인 것 같다.  

다른 프로젝트를 현재 프로젝트에서 쓸 수 있도록 넣어야 했는데, 프로젝트를 옮긴 후 실행해보니 모든 오브젝트가 핫핑크였다.  

보통 이런 경우 Render Pipeline Converter 를 사용하여 셰이더를 URP로 바꿔줄 수 있다.  

![URP1](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-07-25_URP1.png){: width="50%" height="50%"}  

하지만 문제의 셰이더는 Custom Shader라 컨버터가 알아서 변환해줄 수 없는 상태였다. 

<br>

셰이더 코드는 잘 모르는 내겐 날벼락이었다.  

처음 며칠간은 Shader 공부를 하며 어떻게든 코드를 URP 버전으로 다시 짜보려고 했지만...  

코드 양이 많아서 셰이더 초보자인 내가 주어진 R&D기간 안에 해낼 수 없을 것 같았다.  

그러다가 발견한 Custom Shader -> URP 로 변환하는 방법!

![URP2](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-07-25_URP2.png)  

![URP3](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-07-25_URP3.png)  

<mark style='background-color: #fff5b1'>셰이더 코드에서 Tags 에 있는 LightMode를  

<mark style='background-color: #fff5b1'><b>ForwardBase -> UniversalForward</b> 로 변경해주면 된다. </mark>

