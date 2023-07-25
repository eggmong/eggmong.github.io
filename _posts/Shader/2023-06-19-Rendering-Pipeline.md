---
title:  "렌더링 파이프라인 (Rendering Pipeline)"
excerpt: "렌더링의 순서"

categories:
  - Shader
tags:
  - [Shader, Rendering]

toc: false
toc_sticky: false
 
date: 2023-06-20
last_modified_at: 2023-06-20
comments: true
---

> 들어가기에 앞서,  
> 💡 Draw Call 이란?  
> 나중에 따로 글을 작성하겠지만...  
> 쉽게 말해 CPU가 GPU에게 그려라! 라고 명령하는 것이다.  
> 이 그려라! 해서 거치는 과정이 렌더링 파이프라인인 것이다.  

<br>

## 1단계 : 오브젝트 데이터 받아오기

버텍스 정보들을 받아온다.
