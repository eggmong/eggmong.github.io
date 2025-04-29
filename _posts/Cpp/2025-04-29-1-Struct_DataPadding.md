---
title:  "C++ 기초 정리 (구조체 struct, data padding)"

categories:
  - Cpp
tags:
  - [Cpp, C++, struct, data padding]

toc: true
toc_sticky: true
 
date: 2025-04-29
last_modified_at: 2025-04-29
---

<br>

# 구조체
## 데이터 패딩

```cpp
enum class EPlayerJob : unsigned char
{
	None,
	Knight,
	Archer,
	Magicion
};

struct FPlayerInfo
{
	// 문자열의 끝은 항상 0으로 되어 있어야 한다. (NULL) (닫는다는 뜻)
	// 그래서 char 배열은 끝 문자 빼고 총 31개의 문자를 넣을 수 있다.
	char Name[32] = {};

	EPlayerJob Job = EPlayerJob::None;
	int Attack = 0;
	int Defense = 0;
	int Hp = 0;
	int HPMax = 0;
	int MP = 0;
	int MPMax = 0;
	int Exp = 0;
	int Level = 1;
	int Gold = 0;
};
```

위의 구조체 정의부에 마우스를 올려보면, '크기'는 72바이트로 뜨는데 '맞춤'은 4바이트가 뜬다.  
왤까?  
 
'맞춤'이란 즉 <b>메모리의 경계값</b>이란 뜻이다.  
컴파일러는 <b>구조체에서 가장 큰 멤버 기준</b>으로 정렬하려고 하며,  
이 구조체는 4바이트가 최대 정렬 단위이므로 '맞춤'은 4바이트가 된다.  

'크기'는 구조체의 크기인데,  
배열 `Name`의 크기는 1 x 32 = 총 32바이트이고,  
그 다음 오는 enum class 변수 `Job`의 경우 타입이 unsigned char 타입이라 1바이트 이지만,  
다음 멤버가 int이므로 `Job` 뒤에 3바이트 패딩이 들어간다.  
그리고 나머지 int 변수 9개 x 4 = 36바이트 이다.  
여기까지 모두 더하면 32 + (1 + 3) + 36 = 72바이트가 된다.  

만약 `Name` 배열의 크기를 30으로 바꾼다면? (char Name[30])  
구조체의 크기는 68바이트가 된다.  
배열 다음으로 오는 변수 `Job`은 1바이트인데, 정렬 기준인 4바이트에 맞춰  
<b>4의 배수</b>로 메모리가 잡혀 총 32바이트에 `Name` 배열과 `Job` 변수가 들어가게 된다.  
이후 int 변수 크기 36바이트를 모두 합하면 68바이트가 된다.  

```cpp
struct FTest
{
	// 맞춤 1바이트, 크기 1바이트
	char Test1;
};

struct FTest2
{
	// 맞춤 4바이트, 크기 4바이트
	int Test1;
};

struct FTest3
{
	// 맞춤 8바이트, 크기 8바이트
	__int64 Test1;
};

struct FTest4
{
	// 맞춤 8바이트, 크기 16바이트
	int Test1;
	char Test2;
	__int64 Test3;
};
```

들어있는 멤버의 타입에 따라 <mark style='background-color: #fff5b1'>'구조체 멤버 맞춤'</mark>이 일어난다는 것이다.  
`FTest4`의 경우 int 4바이트, char 1바이트 = 5바이트 인데,  
제일 큰 멤버 변수 메모리값이 8바이트라서 (__int64), 8바이트로 정렬 기준이 정해진다.  
그래서 int형, char형 변수들은 8바이트 메모리에 들어가게 된다.  
결국 16바이트로 메모리가 잡히게 된다.  

> 서버와 통신을 할 때, 서버와 클라가 데이터 패딩 규약을 동일하게 맞춰야 한다.  
  클라 입장에선 빈 공간인데, 서버는 패딩으로 공간을 채웠을 경우 통신 시 데이터가 불일치하기 때문에.  



## 빈 구조체의 크기?

```cpp
struct FTest5
{

};
```

> 구조체 안에 멤버 변수를 아무것도 선언 안했을 경우 메모리 크기는?  
  -> 기본으로 <b>1바이트</b>. 최소 바이트 단위.  

구조체는 즉 <mark style='background-color: #fff5b1'>'변수 타입'</mark>이다.  
이 타입을 이용해 변수를 선언할 수 있는데, 0바이트라면 변수를 아예 선언할 수 없으니  
최소 바이트 단위인 1바이트로 메모리를 확보해놓는 것이다.  



## 구조체 초기화

```cpp
void main()
{
	// 구조체 선언 뒤에 = {} 를 해주면, 구조체 멤버 전체를 0으로 초기화 한다.
	FPlayerInfo info = { };
}
```