---
title:  "C++ 기초 정리 (클래스, 생성자 및 소멸자, static과 싱글톤)"

categories:
  - Cpp
tags:
  - [Cpp, C++, 생성자, 소멸자, 복사 생성자, 이동 생성자, Singleton, static]

toc: true
toc_sticky: true
 
date: 2025-05-09
last_modified_at: 2025-05-09
---

<br>

# 클래스

## 클래스의 메모리 크기

클래스의 메모리 크기는 멤버변수의 크기와 동일하다.  
멤버함수는 클래스를 이용하여 생성한 객체의 메모리 크기에 영향을 주지 않는다.  
단, 상속을 사용하면서 가상함수를 사용할 경우 다르게 나온다.  
가상 함수가 있으면 <b>가상 함수 테이블 포인터(V-Table Pointer)</b>가 추가되어 메모리 크기에 영향을 준다.  



## 기본 생성자, 소멸자 (+ this)

생성자와 소멸자는 클래스 이름과 같은 이름의 함수이다.  
반환 타입 없이, 이름과 인자만으로 선언한다.  
생성자는 오버로딩이 가능하지만 소멸자는 인자 없는 형태만 존재한다.  

- 생성자: 이 클래스를 이용해서 객체를 생성할 때 자동으로 호출 되는 함수.  
- 소멸자: 이 클래스를 이용해서 생성한 객체가 메모리에서 제거될 때 자동으로 호출 되는 함수.  


### 클래스에서 this의 의미

클래스의 일반 멤버함수(일반 멤버함수, 생성자, 소멸자) 내부에서는 `this`를 사용할 수 있다.  
this는 <u>자기 자신의 메모리 주소</u>이다.  
`test1.Output();` 이런식으로 클래스의 멤버함수를 호출할 때, 객체를 이용해서 호출하게 되는데  
호출하는 순간 this는 test1의 주소를 가지게 되고,  
`test2.Output();` 으로 호출하면 호출하는 순간 this는 test2의 주소를 가지게 된다.  

> 💡 멤버함수 내부에서 클래스 멤버변수를 사용하게 되면 앞에 `this->` 가 생략된 것이다.  

```cpp
// CTest Class
class CTest
{
public:
	void Output()
	{
		printf("this Test = %p", &this);			// <- &this 는 에러가 남.
	}

	void OutputAttack()
	{
		printf("공격력 : %d\n", this->mAttack);		// this->mAttack 이나, mAttack이나 같은 것.
	}

	int mAttack;
};

int main()
{
	printf("CTest Class Size = %d\n", sizeof(CTest));

	CTest* test = nullptr;
	
	test.Output();								// <- 에러. this는 현재 nullptr 이니까.
	printf("Attack : %d", test.mAttack);		// <- 마찬가지로 에러.
	return 0;
}
```

> 💡 this는 변수가 아님!  
> 만약 변수였다면, CTest 클래스를 `sizeof` 로 크기 구했을 때,  
> this의 변수 크기까지 포함되어 계산되었을 것.  
> (클래스의 메모리 크기는 클래스의 멤버변수의 크기와 동일)  
>
> 만약 클래스 변수를 nullptr 로 초기화 했을 경우,  
> 직접 멤버변수에 접근하면 에러가 남. this는 nullptr 이 되니까.  



### memset : 메모리를 원하는 값으로, 1바이트 단위로 채움

메모리를 원하는 값으로 채워줄 때 사용.  
단, 채우는 단위는 <b>'1바이트 단위'</b>로 값을 채워준다!  
1번 인자의 메모리를, 2번 인자의 값으로, 3번 인자의 크기만큼 채워준다.  
`memset(mArray, 0, sizeof(int) * 10);`  

```cpp
int number = 10;
memset(&number, 1, sizeof(int));	// 겁나 큰 값 나옴.
```

memset은 <b>'1바이트 단위'</b>로 값을 채우게 됨.  
int는 4바이트 이므로 1바이트 단위로 값을 채우게 되면  
| 0000 0001 | 0000 0001 | 0000 0001 | 0000 0001 |  
이렇게 들어가게 되어서.  


```cpp
// main.cpp

struct FPlayer
{
	char	Name[32] = {};
	int		Attack = 0;
};

class CTest
{
public:
    CTest()	:               // 이니셜라이저를 이용해서 멤버변수의 값을 선언과 동시에 초기화할 수 있다.
		mAttack(20),
		mDefense(10)
    {
        // 아래는 mAttack 변수가 선언이 된 후에 50을 대입한다.
        mAttack = 50;

        memset(mArray, 0, sizeof(int) * 10);
        printf("CTest 생성자\n");
    }

    ~CTest()
    {

    }

    // CTest test2("바보"); 이 줄에서 호출되도록 만든 생성자지만, 
    // 이니셜라이저 mName(Name) 또는 생성자 내부에 mName = Name; 으론 수정할 수 없음.
    CTest(const char* Name)	:
	    mAttack(30),
	    mDefense(40)
    {
	    printf("CTest const char* 생성자\n");
    }

    void Output()
    {
        printf("this = %p\n", this);
        printf("공격력 : %d\n", mAttack);
        printf("방어력 : %d\n", this->mDefense);
        //printf("this Addr = %p\n", &this);            // <- 에러.

    }

public:
	// 이 아래로 다른 접근제어 키워드를 만나기 전까지 모두 public이 된다.
	char	mName[32] = {};
	int		mAttack = 0;
	int*	mArray = nullptr;

private:
	int		mDefense = 0;
};

void OutputPlayer(FPlayer* Player)
{
	printf("공격력 : %d\n", Player->Attack);
}

int main()
{
    FPlayer	Player;                         // 구조체 변수 선언
    Player.Attack = 10;
    OutputPlayer(&Player);

    CTest test1, test2("이름");

    printf("test1 = %p\n", &test1);         // test1의 주소 출력
    printf("test2 = %p\n", &test2);         // test2의 주소 출력

    // Output 멤버함수는 test1과 test2 모두 동일한 주소의 함수를
    // 사용하게 된다. 그런데 멤버함수에서 정확하게 test1과 test2의
    // 멤버변수의 값을 인식하고 사용할 수 있다.
    test1.mAttack = 100;
    test1.Output();

    test2.mAttack = 300;
    test2.Output();

    return 0;
}
```



## 복사 생성자, 이동 생성자


- 복사생성자 : 다른 객체를 복사한 새로운 객체를 만들 때 사용한다.  
  `CTest(const CTest& copy)` 형태이다.  
- 이동생성자 : 다른 객체가 가지고 있는 내용을 새로운 객체로 옮겨주면서 객체를 만들 때 사용한다.  
  `CTest(CTest&& other)` 형태이다.  
  자원의 포인터만 복사함 (얕은 복사), RValue 개체를 LValue로 이동한다는 것.  

  - `int& test = number + 10;`		-> 임시객체라서 참조 안 됨.  
  `int&`는 <b>좌값 참조</b>, `number + 10`은 우값(RValue).  
  즉, <u>좌값 참조 <- 우값</u>의 형태이다. 그래서 에러 발생.  
  - `int&& test = number + 10;`		-> 임시객체인데 참조 됨.  
  int&&는 <b>우값 참조</b>. `number + 10`은 RValue. 그래서 참조 가능.  



### memcpy : 메모리를 복사할 때 사용.


memcpy : 메모리를 복사할 때 사용.  
1번 인자의 주소에, 2번 인자의 주소로부터 3번 인자에 들어가는 바이트 수 만큼을 복사해줌.  
`memcpy(mArray, ref.mArray, sizeof(int) * 10);`  


```cpp
// main.cpp

class CTest
{
	// 복사 생성자
	CTest(const CTest& ref)
	{
		//*this = ref;
		// 얕은 복사. ref에 들어온 객체를, this*가 가리키는 변수에 대입해버리는 것임.
		// *this는, main에서 작성한 CTest copyTest(Player1); 에서 copyTest 변수인거임.
		//// 이동 생성자를 구현하면 이 코드는 사용할 수 없음.
		
		mArray = new int[10];
		memcpy(mArray, ref.mArray, sizeof(int) * 10);

		/* 이것보단 memcpy를... 사용하는 게 더 간결할거다.
		for (int i = 0; i < 10; i++)
		{
			mArray[i] = ref.mArray[i];
		}
		*/
	}

	// 이동 생성자
	CTest(CTest&& ref)
	{

	}
};

int main()
{
	CTest test1;						// 기본 생성자 호출
	CTest copyTest(test1);				// 복사 생성자 호출 (LValue 넘김)

	// 임시 객체 생성 후 즉시 소멸됨.
	// 생성자 호출 후 소멸자 호출되어 바로 사라짐. (혹은 컴파일러 최적화 때문에 아무 일도 안일어남남)
	CPlayer();

	// 복사 생성자 호출
	CTest* test2 = new CTest(test1);

	// std::move : 인자에 들어온 객체를 이동시키는 함수.
	// CTest()로 임시 객체 생성 후, std::move 로 RValue로 변환.
	CTest test3 = std::move(CTest());	// 이동 생성자 호출됨

	return 0;
}
```

> 💡 참고: 복사 생성자에서 `*this = ref;` 코드는, 이동 생성자를 구현하면 <u>사용할 수 없게 됨.</u>  
> 이동 생성자가 없으면 컴파일러는 자동으로 복사 생성자, 복사 대입 연산자, 소멸자를 생성해줌.  
> -> 따라서 `*this = ref;` -> `operator=(const CTest&)` 가 자동으로 생성돼서 문제 없음.  
> 하지만 이동 생성자를 구현해버리면,  
> 복사 대입 연산자 `operator=(const CTest&)` 를 자동 생성하지 않음.  
> 그래서 `*this = ref;` -> 에러가 발생한다.  
> 
> 해결 방법: 직접 복사 대입 연산자를 만들어 준다.  
> ```cpp
> CTest& operator=(const CTest& ref)
> {
>    // 멤버 복사
>    return *this;
> }
> ```



# static 키워드와 싱글톤


```cpp
// main.cpp

#include <iostream>

class CStatic
{
	CStatic() { }
	~CStatic() { }

private:
	// 일반 멤버변수는 생성하는 객체마다 메모리를 생성하지만,
	// static 의 경우, 이 클래스를 이용해서 생성한 모든 객체가 공유하는 메모리가 된다.
	// 다만 static 멤버 '변수'는, 클래스 외부에 선언 부분이 필요하다.
	int mNumber = 0;
	static int mNumberStatic;

public:
	int GetNumber() const
	{
		return mNumber;
	}

	static void OutputStatic()
	{
		// mNumber는 static이 아님. 비정적 멤버. 그래서 정적함수에서 호출 안됨.
		// 정확히는, static 멤버함수엔 this가 존재하지 않음. (그치그치.. 모든 객체가 공유하니까)
		// 그래서 CStatic static1; 하고 static1->OutputStatic(); 이나, CStatic::OutputStatic(); 이나 같음.
		// 다만 특정 static 객체의 멤버변수 출력하고 싶다면 - CStatic::OutputObject(&static1); 이렇게.
		//printf("Number : %d\n", mNumber);
	}
};

// 클래스의 static 멤버변수 선언.
int CStatic::mNumberStatic;

int main()
{
	// ~~
	return 0;
}
```



## 기본적인 싱글톤 클래스


```cpp
class CSingleton
{
public:
	CSingleton() { }
	~CSingleton() { }

	static CSingleton* GetInstance()
	{
		if (mInstance == nullptr)
		{
			mInstance = new CSingleton();
		}

		return mInstance;
	}

	static void Destroy()
	{
		if (mInstance)
		{
			delete mInstance;
			mInstance = nullptr;
		}
	}

private:
	static CSingleton* mInstance;
};


CSingleton* CSingleton::mInstance = nullptr;


int main()
{
	CSingleton::GetInstance();

	CSingleton::Destroy();

	return 0;
}
```