---
title:  "C++ 기초 정리 (함수 포인터)"

categories:
  - Cpp
tags:
  - [Cpp, C++, Method Pointer]

toc: true
toc_sticky: true
 
date: 2025-05-01
last_modified_at: 2025-05-01
---

<br>

# 함수 포인터

함수 포인터 : 함수는 메모리의 코드 영역에 저장된다.  
- 함수 주소는 <b>코드 영역에 저장된 함수의 메모리 주소</b>를 의미한다.  
- 함수 포인터는 이런 함수 주소를 저장할 수 있는 변수 타입이다.  
- `반환타입 (*포인터변수명) (인자타입들);` 의 형태로 선언한다.  
- 함수 타입은 <b>반환타입, 인자</b>에 영향을 받는다.  
- 반환 타입이나 인자가 다르면 다른 타입으로 인식한다.  

```cpp
int Sum(int num1, int num2)
{
	return num1 + num2;
}

int Minus(int num1, int num2)
{
	return num1 - num2;
}

// 위의 두 함수는 같은 타입이라고 볼 수 있다.



int main()
{
	// 함수 포인터 선언 방법
	// 반환타입 (*포인터변수명) (인자타입들); 의 형태로 선언한다.

	int (*Func1)(int, int) = nullptr;
	void (*Func2)() = nullptr;
	void (*Func3)(int) = nullptr;

	printf("Size = %d\n", sizeof(Func1));				// 포인터이므로 x64 환경에서 8바이트

	// 함수포인터 변수 이름만 빼면 타입이 된다.
	printf("Size = %d\n", sizeof(int(*)(int, int)));	

	// 함수 이름은 곧 함수 주소이다. (마치 배열처럼)
	printf("Address = %p\n", Sum);

	// 이 말은 즉 함수 이름으로 포인터 변수에 대입해줄 수 있고
	Func1 = Sum;

	// 함수의 호출은 함수주소(인자 전달) 의 형태로 호출된다는 것이다.
	printf("Func1 Result = %d\n", Func1(10, 20));

	Func1 = Minus;
	printf("Func1 Result = %d\n", Func1(10, 20));

	

	// 함수 포인터도 배열 선언이 가능하다.
	int (*Func1Array[10])(int, int);

	Func1Array[0] = Sum;
	Func1Array[1] = Minus;

	return 0;
}