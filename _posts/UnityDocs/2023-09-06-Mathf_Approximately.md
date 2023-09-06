---
title:  "Mathf.Approximately : 소수점 값 같은지 확인할 때"
expert: ""

categories:
  - UnityDocs
tags:
  - [Unity, Game Engine, Mathf, float, double]

toc: false
toc_sticky: true
 
date: 2023-09-06
last_modified_at: 2023-09-06
---

<br>

# Mathf.Approximately

두 부동 소수점 숫자가 거의 같은지 확인할 수 있는 함수이다.  

부동 소수점 숫자는 정확한 비교가 어렵기 때문에, == 연산자를 사용할 수 없다.  
대신 Mathf.Approximately를 사용하여 비교할 수 있다.  

```c#
public static bool Approximately(float a, float b);
```

이렇게 정의되어 있고, 여기서 a 와 b는 비교하려는 각 숫자이다.  
a와 b가 거의 같으면 true를 반환하고, 그렇지 않으면 false를 반환한다.  

두 숫자 간의 차이가 작은 경우에도 true를 반환한다.  
이 함수를 사용함으로써 부동 소수점 정밀도의 한계로 인한 비교 오차를 줄여준다.  

```c#
if (Mathf.Approximately(_Pos.x, 1f))
{
  ...
}
```

나는 이런 식으로 사용했다.  