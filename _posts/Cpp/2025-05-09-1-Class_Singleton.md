클래스에서 this의 의미
생성자
복사생성자
이동생성자
소멸자
static (+ 싱글톤)

# 클래스

## 클래스의 메모리 크기

클래스의 메모리 크기는 멤버변수의 크기와 동일하다.  
멤버함수는 클래스를 이용하여 생성한 객체의 메모리 크기에 영향을 주지 않는다.  
단, 상속을 사용하면서 가상함수를 사용할 경우 다르게 나온다.  



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


### memset

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










```cpp
class CPlayer
{
	/*
	생성자와 소멸자는 클래스 이름과 같은 이름의 함수이다.
	반환 타입 없이 이름과 인자만으로 선언한다.
	생성자는 오버로딩이 가능하지만 소멸자는 인자 없는 형태만 존재한다.
	생성자 : 이 클래스를 이용해서 객체를 생성할 때 자동으로 호출이 되는
	함수이다.
	소멸자 : 이 클래스를 이용해서 생성한 객체가 메모리에서 제거될 때 자동으로
	호출이 되는 함수이다.
	*/
public:
	// 생성자
	// 이니셜라이저를 이용해서 멤버변수의 값을 선언과 동시에 초기화할 수 있다.
	CPlayer()	:
		mAttack(20),
		mDefense(10)
	{
		// 아래는 mAttack 변수가 선언이 된 후에 50을 대입한다.
		mAttack = 50;

		// 메모리를 원하는 값으로 채워줄 때 사용
		// 단, 채우는 단위는 '바이트 단위'로 값을 채워준다!
		// 1번 인자의 메모리를, 2번 인자의 값으로, 3번 인자의 크기만큼 채워준다.
		memset(mArray, 0, sizeof(int) * 10);
		printf("CPlayer 생성자\n");
	}

	CPlayer(const char* Name)	:
		mAttack(30),
		mDefense(40)
	{
		printf("CPlayer const char* 생성자\n");
	}

	/*
	복사생성자 : 다른 객체를 복사한 새로운 객체를 만들 때 사용한다.

	이동생성자 : 다른 객체가 가지고 있는 내용을 새로운 객체로 옮겨주면서
	객체를 만들 때 사용한다.
	클래스타입&&
	RValue 타입이라고 한다.

	int& test = number + 10;		// 임시객체 참조 안 됨
	int&& test = number + 10;		// 임시객체 참조 됨
	*/

	/*
	얕은복사: 클래스의 데이터를 그대로 복사한다. 주소값까지 동일하게. 그래서 복사한 여러 객체가 있을 때, 모두 삭제한다면-... 삭제한 걸 또 삭제하게 되는 일 발생
	깊은복사: 
	*/

	CPlayer(const CPlayer& ref)
	{
		// 얕은 복사. ref에 들어온 객체를, this*가 가리키는 변수에 대입해버리는 것임.
		// this*는, 밑에서 작성한 .. CPlayer copyTest(Player1); 에서 copyTest 변수인거임.
		//*this = ref;

		mArray = new int[10];

		// 메모리를 복사할 때 사용.
		// 1번 인자의 주소에, 2번 인자의 주소로부터 3번 인자에 들어가는 바이트 수 만큼을 복사해줌.
		// memcpy
		memcpy(mArray, ref.mArray, sizeof(int) * 10);

		/* 이것보단 memcpy를...
		for (int i = 0; i < 10; i++)
		{
			mArray[i] = ref.mArray[i];
		}
		*/
	}

	// 이동생성자. 이거 생성하는 순간, 위에 복사생성자에서 얕은 복사로 썼던 *this = ref; 를 못쓰게 됨.
	// 생성자 호출되고 이러는 부분들 면접에 많이 나온단다.
	CPlayer(CPlayer&& ref)
	{

	}

	// 소멸자
	~CPlayer()
	{
		printf("CPlayer 소멸자\n");
	}

public:
	// 이 아래로 다른 접근제어 키워드를 만나기 전까지 모두 public이 된다.
	char	mName[32] = {};
	int		mAttack = 0;
	int*		mArray = nullptr;

private:
	int		mDefense = 0;

public:
	// 클래스의 일반 멤버함수(일반멤버함수, 생성자, 소멸자) 내부에서는
	// this를 사용할 수 있다.
	// this는 자기 자신의 메모리 주소이다.
	// Player1.Output() 이런식으로 클래스의 멤버함수를 호출할 때 객체를
	// 이용해서 호출하게 되는데 Player1.Output() 으로 호출하면 호출하는
	// 순간 this는 Player1의 주소를 가지게 되고 Player2.Output() 으로
	// 호출하면 호출하는 순간 this는 Player2의 주소를 가지게 된다.
	// 멤버함수 내부에서 클래스 멤버변수를 사용하게 되면 앞에 this-> 가 
	// 생략된 것이다.
	void Output()
	{
		printf("this = %p\n", this);
		printf("공격력 : %d\n", mAttack);
		printf("방어력 : %d\n", this->mDefense);
		//printf("this Addr = %p\n", &this);
	}

	void OutputTest()
	{
		printf("OutputTest Function\n");
	}
};

void OutputPlayer(FPlayer* Player)
{
	printf("공격력 : %d\n", Player->Attack);
}

int main()
{
	FPlayer	Player;

	Player.Attack = 10;

	OutputPlayer(&Player);

	CPlayer	Player1, Player2("이름");

	printf("Player1 = %p\n", &Player1);
	printf("Player2 = %p\n", &Player2);

	// Output 멤버함수는 Player1과 Player2 모두 동일한 주소의 함수를
	// 사용하게 된다. 그런데 멤버함수에서 정확하게 Player1과 Player2의
	// 멤버변수의 값을 인식하고 사용할 수 있다.
	Player1.mAttack = 100;
	Player1.Output();

	Player2.mAttack = 300;
	Player2.Output();

	CPlayer* Player3 = nullptr;

	//Player3->Output();
	Player3->OutputTest();

	printf("Player Size = %d\n", sizeof(CPlayer));

	CPlayer copyTest(Player1);

	int number = 10;
	memset(&number, 1, sizeof(int));	// 겁나 큰 값 나옴.
	// memset은 1바이트 단위로 값을 채우게 됨.
	// int는 4바이트 이므로 1바이트 단위로 값을 채우게 되면
	// 0000 0001 0000 0001 0000 0001 0000 0001 ... 이렇게 들어가게 되어서. 

	// 임시 객체 생성. 이 줄에서 생성자 호출되고, 줄 넘어가기 직전 소멸자 호출되며 사라짐.
	CPlayer();

	CPlayer* test2 = new CPlayer(Player1);

	// std::move : 인자에 들어온 객체를 이동시키는 함수.
	// CPlayer test2 = std::move(CPlayer());	<- 이동생성자 호출됨


	return 0;
}
```


static, singleton
```
#include <iostream>

class CStatic
{
	CStatic()
	{

	}

	~CStatic()
	{

	}

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


class CSingleton
{
public:
	CSingleton()
	{

	}

	~CSingleton()
	{

	}

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

public:


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