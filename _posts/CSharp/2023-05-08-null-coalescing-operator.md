---
title:  "null 병합 연산자 (?? 및 ??= ...물음표 두개 연산자)"

categories:
  - CSharp
tags:
  - [CSharp]

toc: true
toc_sticky: true
 
date: 2023-05-09
last_modified_at: 2023-05-09
---

Nullable 에 대해서 글을 작성했다가 생각난 김에 기억해두려고 쓰는 글.  
그동안 정확한 이름을 기억하지 못해서 매번 물음표 두 개 연산자라고 불렀었는데 ㅋㅋ  
이번에 글 쓰면서 기억하게 되었다.

## null 병합 연산자

널 병합 연산자(??)는 앞의 값이 null이 아니라면 앞의 값을 반환하고, 뒤의 값은 체크하지 않는다.  
앞의 값이 null이라면 뒤의 값을 체크하여 그 결과를 반환한다.


## null 병합 연산자

널 병합 할당 연산자(??=)는 앞의 값이 null이라면 뒤의 값을 앞의 피연산자 변수에 할당한다.  
이 역시 앞의 값이 null이 아니라면 뒤의 값은 체크하지 않는다.

<br>

```c#
int? a = null;      // Nullable a : null

// null 병합 연산자
int b = a ?? 1;     // b : 1

// null 병합 할당 연산자
a ??= 100;          // a : 100
```

***  
참고 링크
- [https://learn.microsoft.com/ko-kr/dotnet/csharp/language-reference/operators/null-coalescing-operator](https://learn.microsoft.com/ko-kr/dotnet/csharp/language-reference/operators/null-coalescing-operator)