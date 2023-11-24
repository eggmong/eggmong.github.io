---
title:  "참조 (Reference)"

categories:
  - Cpp
tags:
  - [Cpp, C++, 참조]

toc: true
toc_sticky: true
 
date: 2023-11-24
last_modified_at: 2023-11-24
---

<br>

> 🤯 언리얼을 하기 위해 C++ 기억 되살리기 프로젝트

# 참조 (Reference)

매개변수를 전달할 때 값을 복사해서 전달할거냐, 메모리를 참조해서 전달할거냐.  
포인터 연산이 없어도 될 때, 주소 변경이 없어도 될 때 안전하게 쓰라고 만들어진 것이 참조!  

- 별칭이다
```cpp
int number = 10;
int& ref = number;	// number를 다른 이름으로 부르는 것.
```

- NULL 이 될 수 없음.
```cpp
int& ref = NULL;	// error
```

- 초기화 중에 반드시 선언되어야 함.
```cpp
int& ref;			//error 선언할 때 반드시 나는 누구의 별칭이다~ 이렇게 선언이 되어야 함.
```

- 참조하는 대상을 바꿀 순 없음
```cpp
int number1 = 10;
int number2 = 20;
int& ref = number1;		// number1를 참조
ref = number2;		// 참조 대상 변경이 아니라 number2값의 대입이 일어나게 됨
// => 세 변수 모두 값이 20이 됨
```

## 함수 매개변수로서의 참조

- 포인터 ver.
```cpp
void swap(int* number1, int* number2)
{
	int temp = *number1;
	*number1 = *number2;
	*number2 = temp;
}
```

- 참조 ver.
```cpp
void swap(int& number1, int& number2)	// 매개변수로 넣어주는 변수들을 복사해오는 게 아니라 그걸 참조한다는 것
{
	int temp = number1;		// number1이 참조하고 있는 변수의 값을 대입
	number1 = number2;
	number2 = temp;
}
```

> ✨ 참조 버전으로 작성하면 좋은 점?  
> number1, number2의 변수는 NULL이 될 수 없음. (위에 작성했던 참조 특성)  
> &nbsp;&nbsp;👉 포인터의 경우 NULL 넣으면 프로그램 뻑남. 참조 버전의 경우 NULL 체크 안해도 됨.  
> 소유하지 않은 메모리를 가리킬 수 없음.  
> &nbsp;&nbsp;👉 포인터 연산의 경우 메모리 연산을 할 수 있는데(좋지 않지만), 참조는 불가능하니까  

## 컴퓨터는 참조를 알아들을까?

- 모름. 어셈블리단에선 동일함. 포인터와 참조는 같은 어셈블리 명령어를 생성함.  
- 컴파일러는 참조를 포인터로 바꿔 줌. 기계가 이해할 수 있도록