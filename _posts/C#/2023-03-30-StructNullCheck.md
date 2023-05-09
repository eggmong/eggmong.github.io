---
title:  "Struct 타입 null check (Nullable Type)"

categories:
  - C#
tags:
  - [C#]

toc: true
toc_sticky: true
 
date: 2023-03-30
last_modified_at: 2023-05-08
---

struct 타입 변수에 값이 들어가있는지 확인해야 했다.

먼저 struct는 값 타입이라 null이 될 수 없다.  
디폴트 값이든 '값'이 들어가 있다.  
그래서 일반적인 null check로는 알 수 없다.

default 값이 들어가 있는지 struct 요소들 값들을 일일히 비교해줘도 되지만...  
```c#
//구조체 변수 myStruct에 대한 값 비교  

if (myStruct.X == 0 || myStruct.Y == 0)
```
이보다 좀 더 간단하게 끝낼 수 있는 방법이 없나 찾아봤다.  

## 1. Equals(default(T))
```c#
if (myStruct.Equals(default(MyStruct)))
```
말그대로 구조체 변수의 값들이 struct의 디폴트 값과 같은지 비교하는 것이다.

하지만 구조체의 디폴트 값이 0일 때 position 값을 담고 있는 구조체라면,  
position 에선 0 또한 유효한 값이 될 수 있기 때문에 명확한 비교 방식이 될 수 없다.  

## 2. Nullable 타입 사용
```c#
int? temp = null;
MyStruct? myStruct = null;
```
Nullable 타입은 값 타입도 Null Check를 할 수 있도록 (null을 할당할 수 있도록) 필드를 지원해주는 구조체이다.  
>구조체이므로 즉 Nullable 은 <b>값 타입</b>이다.  
  
    
```c#
int? temp1 = null;
Nullable<int> temp2 = null;
```
<b>'?'</b> 물음표 키워드로 Nullable 타입을 축약해서 쓸 수 있다.  
위의 코드처럼 int 뒤에 물음표를 붙이면, 해당 변수는 Nullable int형 타입임을 의미한다. 그래서 null을 할당할 수 있게 된다.  

Nullable은 null check를 할 수 있게 해주는 HasValue 필드와, 실제 값을 담고있는 Value 필드를 제공한다. 
```c#
int? temp3 = null;
if (temp3.HasValue)
{
  Debug.Log(temp3.Value);
}
```



***  
참고 링크
- [https://learn.microsoft.com/ko-kr/dotnet/api/system.nullable-1?view=net-7.0](https://learn.microsoft.com/ko-kr/dotnet/api/system.nullable-1?view=net-7.0)