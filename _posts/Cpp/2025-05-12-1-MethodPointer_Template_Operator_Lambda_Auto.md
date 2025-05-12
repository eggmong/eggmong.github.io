---
title:  "C++ 기초 정리 (함수 포인터, 템플릿, function, 람다, auto, 오퍼레이터)"

categories:
  - Cpp
tags:
  - [Cpp, C++, pointer, template, function, lambda, auto, operator]

toc: true
toc_sticky: true
 
date: 2025-05-09
last_modified_at: 2025-05-09
---

<br>

# 클래스의 함수 포인터


## 전역/정적 함수 포인터


`반환타입 (*포인터이름)();` 으로 선언해준다.  

```cpp
void Output() { printf("어쩌고저쩌고"); }

void (*Func)();				// <- 함수 포인터 선언.  
Func = Output;` 			// 가리킬 함수 넣어주기.  
// ...
```


## 멤버함수 포인터

전역/정적 함수와, 멤버 함수 포인터를 선언해주는 방식이 다르다.  
전역/정적의 경우 호출할 때 this가 붙지 않음. (객체를 통해서 호출하지 않음)  

멤버함수의 경우 this가 붙어 있으므로, 참조할 타입을 앞에 넣어줘야 한다.  
`반환타입(클래스명:: * 이름) () = &클래스명::함수명` 형태로 넣어줘야 한다.  

```cpp
// 멤버함수 포인터 생성

// CTest 클래스 정의 생략
CTest test;
void(CTest:: * Func1) () = &CTest::Output;
(test.*Func1)();
// ...
```


## 정리


```cpp
// main.cpp

class CTest
{
public:
	// 생성자, 소멸자 생략

	void Output() { }					// 멤버함수
	static void OutputStatic() {}		// 정적 멤버함수
};

void Output()
{
	// 전역함수
}

struct FTestFunc
{
	// 이런식으로 구조체로 미리 선언해서 사용할 수도 있음.
	// 하지만 매번 작성하기 번거로움. 그래서 템플릿 사용.
	void(CTest::* Func1)();
	CTest* Object;
};

int main()
{
	CTest test;

	// 함수 포인터 선언.
	void (*Func)();

	// 전역함수는 this가 없음. 함수 포인터에 바로 담을 수 있음.
	Func = Output;

	// static 함수는 객체로도 접근 가능하고 (test.OutputStatic();)
	// 클래스명으로도 접근 가능함. 즉 this가 없음. 그래서 역시 바로 담을 수 있음.
	Func = CTest::OutputStatic;

	Func();

	//Func = CTest::Output(); 에러. CTest의 Output은 멤버함수이다.
	// 
	// 일반 멤버함수 포인터는 &클래스명::함수명 으로 해야 한다.
	// 멤버함수 호출할 때 어느 객체인지 가리키는 this가 붙음. 즉 this가 있으므로 그에 대한 지정을 해줘야 함.
	void(CTest:: * Func1) () = &CTest::Output;

	// 멤버함수 포인터 호출 방법
	// (객체.*멤버함수포인터) (인장); 의 형태로 호출한다.
	
	//Func1();	// 에러. 0개 인수를 받아들이는 어쩌고저쩌고... 
	// 하여튼 어느 객체를 통해 함수포인터 호출할건지 써야함.
	(test.*Func1)();
}
```




# 템플릿


가변적인 타입에 대응하기 위해 사용.
컴파일 시간에 타입이 결정되며, 결정된 타입으로 코드를 만들어준다.  
`template <typename 원하는 이름>` 으로 만들 수 있다.  
`template <class 이름>` 의 형태로도 만들 수 있다.  
`template <typename T, typename T1>` <- T와 T1은 타입이 된다.  


```cpp
// main.cpp

#include <functional>				// 템플릿 사용하기 위한 헤더.
									// 멤버함수건, 전역함수건 모든 함수를 포인터에 저장할 수 있도록
									// 지원해주는 기능이 있음

template <typename T>
void OutputTemplate(T num)
{
	std::cout << num << std::endl;
}

template <typename T>
void OutputTemplateFunc(T Func)
{
	Func();
}

// 클래스와 함수 포인터를 받도록 설계
template <typename T, typename T1>
void OutputObject(T* name, T1 name2)
{
	(name.*name2) ();
}


int main()
{
	OutputTemplate<int>(10);

	// T가 함수포인터 타입이 되는 것. 그래서 함수 호출 가능.
	OutputTemplate<void(*)()>(Output);

	OutputObject<CTest, void(CTest::*)()>(&test, &CTest::Output);

	return 0;
}

```



# functional

## std::bind

- 전역/정적 함수 호출 할 경우  
  1번 인자: 함수 주소, 2번 인자부터는 주소를 넣어주는 함수의 인자를 어떤 순서로 호출해줄지 결정한다.  
  `func2 = std::bind(OutputFunctional, std::placeholders::_1, std::placeholders::_2);`  
- 멤버 함수 호출 할 경우  
  인자가 없을 때, 멤버 함수를 넣는다면 -> 1번인자: 함수 주소, 2번인자: 객체  
  `std::function<void()> func3;`  
  `func3 = std::bind(&CTest::Output, test);`  



```cpp
// main.cpp
#include <functional>

// CTest 클래스는 위의 코드와 같음

void OutputFunctional(int num1, float num2)
{
	std::cout << num1 << ", " << num2 << std::endl;
}

int main()
{
	CTest test;

	std::function<void(int, float)> func2;
	// 1번 인자: 함수 주소, 2번 인자부터는 주소를 넣어주는 함수의 인자를 어떤 순서로 호출해줄지 결정한다.
	func2 = std::bind(OutputFunctional, std::placeholders::_1, std::placeholders::_2);
	func2(10, 3.14f);				// 10, 3.14 출력

	func2 = std::bind(OutputFunctional, std::placeholders::_2, std::placeholders::_1);
	func2(10, 3.14f);				// 3, 10  출력.
	// placeholders의 순서에 따라 결정되므로 3.14가 먼저 들어가는데 1번 인자는 int형이라서 잘림.

	// function에 전역함수 넣을 때랑, 멤버함수 넣을 때랑 다름. 위는 전역함수 넣을 때.
	// 인자가 없을 때, 멤버 함수를 넣는다면
	// 1번인자: 함수 주소, 2번인자: 객체
	std::function<void()> func3;
	func3 = std::bind(&CTest::Output, test);

	// 정적 함수 넣는다면 객체 필요 없음.
	func3 = std::bind(CTest::OutputStatic);
	func3();
	return 0;
}
```



# 람다, auto

- 람다: 익명 함수. []() { /*함수 내용*/ };  으로 정의한다. 반환 타입은 `->` 를 넣어준다.  
  `auto func6 = [](int num, int num1) -> int { return num + num1; };`
- auto: 대입받는 값의 타입으로 자동으로 타입이 결정된다. (느림)  


```cpp
int main()
{
	auto num = 10;
	auto func4 = []()
		{
			printf("func4 는 람다. \n");
		};
	func4();										// func4는 람다. 출력

	auto func5 = [](int num)
		{
			printf("func5도 람다 %d\n", num);		// func5도 람다 500 출력
		};
	func5(500);


	auto func6 = [](int num, int num1) -> int		// 반환 타입은 -> 로 표시한다.
		{
			return num + num1;
		};
	printf("%d\n", func6(1, 2));

	// 람다의 캡쳐기능 : 외부 변수 갖다 쓰기
	int num3 = 10, num4 = 20;
	auto func7 = [num3, num4]()
		{
			return num3 + num4;
		};
	printf("func7 : %d\n", func7);					// 인자 안 넣고도 함수 호출 가능함.

	// 외부변수 참조 형태로 갖다쓰기
	auto func8 = [&num3, &num4]()
		{
			++num3;
			++num4;
			return num3 + num4;
		};
	printf("num3 : %d, num4 : %d\n", num3, num4);		// 10, 20 출력
	printf("func8 : %d\n", func8);		// 32 출력



	std::function<void()> func9 = []()
		{
			printf("Test Func9\n");
		};
	func9();											// 람다를 function 객체에 넣어서 호출도 가능.



	// 람다 타입을 추론해서 선언.
	//decltype 어쩌고저쩌고
	
	return 0;
}
```


### decltype: 타입 추론

람다 타입도 추론 가능...  

```cpp
int cnt = 100;
int& refcnt = cnt;

// 컴파일러는 decltype(refcnt) = int &로 추론한다.
decltype(refcnt) decl_refcnt = refcnt;

auto lambda = [](int x) { return x * 2; };
decltype(lambda) another = lambda;  // OK: lambda 타입을 그대로 추론해서 사용
```



# 오퍼레이터

operator : 연산자 재정의 기능이다.  
구조체나 클래스에서 활용할 수 있으며 원하는 연산자를 재정의하여,  
구조체나 클래스에 연산자를 이용할 수 있게 해준다.  

`반환타입 operator 연산자 (오른쪽에 연산될 값) { }`  

만약 10 + 20 일 경우, 오른쪽에 연산될 값은 20이다.  


```cpp
// main.cpp

class COperator
{
public:
	COperator() { }
	~COperator() { }

	COperator(int Num)	:
		mNumber(Num)
	{
		printf("생성\n");			// 값을 받아 대입해주는 생성자
	}

private:
	int	mNumber = 100;

public:
	void operator = (int Num)
	{
		printf("대입 오퍼레이터\n");
		mNumber = Num;
	}

	void operator = (float Num)		// 오퍼레이터 오버로딩도 가능하다는 것.
	{
		mNumber = Num;			
	}

public:
	void Output()
	{
		printf("Number = %d\n", mNumber);
	}

	void OutputConst()	const
	{
		printf("Number = %d\n", mNumber);
	}

	COperator operator + (const COperator& oper)	const
	{
		COperator	result;
		result.mNumber = mNumber + oper.mNumber;

		return result;
	}

	COperator operator + (int Num)	const
	{
		COperator	result;
		result.mNumber = mNumber + Num;

		return result;
	}

	COperator Add(int Num)	const
	{
		COperator	result;
		result.mNumber = mNumber + Num;

		return result;
	}

	// ++이 왼쪽에 오는 전위연산.
	void operator ++ ()
	{
		++mNumber;
	}

	// ++이 오른쪽에 오는 후위연산.
	void operator ++ (int)
	{
		++mNumber;
	}
};

class CArray
{
public:
	CArray() { }
	~CArray() {	}

private:
	int		mArray[10] = {};

public:
	// 반환 값이 참조니까, 값을 바꿀 수 있음.
	int& operator [] (int Index)
	{
		return mArray[Index];
	}

	const int& operator [] (int Index)	const
	{
		return mArray[Index];
	}
};

class CPointer
{
public:
	CPointer() { }
	CPointer(COperator* pt) :
		mPointer(pt)
	{
	}

	~CPointer() { }

public:
	COperator* mPointer = nullptr;

public:
	COperator* operator -> ()
	{
		return mPointer;
	}

	const COperator* operator -> ()	const
	{
		return mPointer;
	}
};

template <typename T>
class CArrayTemplate					// 동적 배열 생성하기.
{
public:
	CArrayTemplate()
	{
		mArray = new T[mCapacity];
	}

	~CArrayTemplate()
	{
		delete[] mArray;
	}

private:
	T* mArray = nullptr;
	int	mSize = 0;
	int	mCapacity = 2;

public:
	T& operator [] (int Index)
	{
		return mArray[Index];
	}

	const T& operator [] (int Index)	const
	{
		return mArray[Index];
	}
};

int main()
{
	// int타입을 인자로 가지고 있는 생성자가 호출된다.
	COperator		oper = 30;

	// 위에서 오퍼레이터로 정의한 대입 연산자가 호출된다.
	oper = 500;

	oper = 3.14f;

	oper.Output();

	COperator	oper1 = 40, result;

	result = oper + oper1;

	result.Output();						// 40 + 3(3.14 짤림) 해서 43 출력.

	result = oper + 200;

	result.Output();

	result = oper.Add(300);

	result.Output();

	CArray	Arr;

	Arr[0] = 30;

	const CArray	cArr;
	//cArr[0] = 50;							// const 반환 타입의 오퍼레이터를 작성해야 사용 가능함.
	int Num = cArr[0];

	CPointer	pt;
	pt.mPointer = &oper;

	// -> 연산자를 호출하여 COperator* 를 반환하기 때문에
	// mPointer-> 가 된다.
	pt->Output();

	const CPointer	cpt = &oper;

	cpt->OutputConst();

	++oper;									// 포인터 연산도 할 수 있다.

	oper++;

	CArrayTemplate<int>		IntArr;
	CArrayTemplate<float>	FloatArr;

	return 0;
}
```