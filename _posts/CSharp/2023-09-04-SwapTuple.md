---
title:  "두 변수의 값 맞바꾸기 (Swap)"
expert: "Tuple 형식"

categories:
  - CSharp
tags:
  - [CSharp]

toc: false
toc_sticky: true
 
date: 2023-09-02
last_modified_at: 2023-09-02
---

<br>

두 값을 맞바꾸는 방법으로는 임시 변수를 만들어 값을 넣어준 다음 바꿔주는 방식만 알고 있었다.  

```c#
var a = 111;
var b = 222;
var temp = a;
a = b;
b = temp;
```

이런 식으로 말이다.  


그런데 이번에 새로운 방식을 알게 되었다.  

```c#
(x, y) = (y, x);
```

튜플 형식을 이용한 Swap 이란다.  
최근에 알게 되어서 블로그에 써둔다.  

<b>* 참고한 링크</b>  
- [https://learn.microsoft.com/ko-kr/dotnet/fundamentals/code-analysis/style-rules/ide0180](https://learn.microsoft.com/ko-kr/dotnet/fundamentals/code-analysis/style-rules/ide0180)  
- [https://hyr.mn/swapping-numbers/](https://hyr.mn/swapping-numbers/)  