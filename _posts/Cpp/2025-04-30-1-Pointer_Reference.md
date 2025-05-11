---
title:  "C++ 기초 정리 (포인터, 이중 포인터, 레퍼런스)"

categories:
  - Cpp
tags:
  - [Cpp, C++, pointer, reference]

toc: true
toc_sticky: true
 
date: 2025-04-30
last_modified_at: 2025-04-30
---

<br>

# 포인터


- 포인터란?
  - 메모리 주소를 저장하기 위한 변수 타입  
  - 모든 변수 타입은 포인터 타입 변수를 선언할 수 있다.
  - 변수타입에 영향을 받아 해당 타입 변수의 메모리 주소를 저장할 수 있다.
  - 변수타입* 포인터변수이름;





## & 연산자

> 변수 앞에 &를 붙여주면 <u>해당 변수의 메모리 주소</u>를 의미한다.  

```cpp
int number = 100;
float number1 = 3.14f;

// %x : 16진수 출력
// %8x : 16진수 8자리 출력
// %p : 메모리 주소 출력
printf("number Address: %p\n", &number);

int* address = nullptr;

// address = &number1;							      // -> 에러
// float 타입 변수의 주소를 int* 타입에 대입해주기 때문에 에러 발생
// 아래와 같이 형변환을 하면 가능하다.
// address = (int*)&number1;

address = &number;

printf("&number = %p\n", &number);				// number 주소값 출력
printf("address = %p\n", address);				// number 주소값 출력
printf("*address = %d\n", *address);			// number의 값 100 출력

// 출력해보면, 숫자 16자리가 출력된다. 16진수는 8자리로 출력되는데.
// 운영체제가 64비트면 16자리, 32비트면 8자리로 출력된다.
// 포인터 변수의 크기가 64비트면 8바이트, 32비트면 4바이트로 생성된다는 소리이다.
// 즉 포인터 변수는 고정적이지 않다.
// 포인터타입의 메모리 크기는 x64일 때 8바이트, x86일 때 4바이트이다.
// 자세한 정보 찾아보기.
```

* [nullptr 관련 포인터 글(링크)](https://eggmong.github.io/cpp/2-nullptr-enum_class/)은 이전에 강의 들어둔 게 있음. 참고.  



### void* 타입

void는 타입이 없다는 의미라 일반 void타입 변수는 선언이 불가능하다.  
그렇지만 void* 타입 변수는 선언이 가능하여, <u>모든 변수 타입의 메모리 주소를 저장할 수 있다</u>.  
단 void* 는 <b>역참조가 불가능</b>하다.  
(역참조: 포인터 변수가 가지고 있는 메모리 주소에 접근하여 해당 주소에 있는 값을 변경하거나,  
가져와서 사용하는 것)  

```cpp
// void 포인터는 모든 타입의 변수를 참조할 수 있지만, 역참조가 불가능

int number = 100;
int* address = nullptr;
address = &number;
*address = 1234;								      // -> 역참조. 참조하던 number 변수의 값을 변경한다.

bool test = false;
bool* testPtr = &test;

// void*
void* voidAddress = &number;					// int형 변수 참조
voidAddress = &test;							    // bool형 변수 참조

// *voidAddress = 100;							  -> 에러. 역참조 발생.

*((bool*)voidAddress) = true;					// void* 를 임시로 bool* 로 변환시켜 대입하기 때문에, 이 경우는 역참조 가능.
```





## 포인터와 배열의 관계

> <b>배열 이름</b>은 <b>배열의 시작 메모리 주소</b>를 의미한다.  

```cpp
void main()
{
  int array[100] = {};
  printf("%p\n", array);						              // array 배열의 메모리 주소 출력

  // 배열의 '이름'은 배열의 '시작 메모리 주소'이기 때문에
  // 포인터 변수에 배열의 이름을 저장할 수 있다.
  int* arrayPtr = array;			
  
  printf("arrayPtr = %p\n", arrayPtr);						// array 배열의 메모리 주소 출력
  
  *arrayPtr = 1;
  printf("array[0] = %d\n", array[0]);	          // 윗 줄에서 array[0]에 1이 대입해져 1 출력.


  // 즉 배열이름[인덱스]로 접근하는 것은, 배열시작주소[인덱스]와 같은 뜻이다.
  array[3] = 500;								                  // array[3] 요소에 500 대입.

  // 포인터 변수가 배열의 시작 메모리 주소를 가지고 있기 때문에, 인덱스를 이용하여 배열에 접근할 수 있다.
  arrayPtr[3] = 1111;							                // array[3] 요소에 1111이 새로 대입됨.
  printf("array[3] = %d\n", arrayTest[3]);	// 1111 출력.

  
  int number = 5;
  ChangeNumber(&number);
  // number의 값이 5가 아닌 5000으로 변경된다.

  int numberArray[25] = {};
  Calculate(numberArray);
  // 이후 numberArray 를 반복문 돌려 출력해보면, Calculate에서 대입되었던 값들이 출력됨.
}

void ChangeNumber(int* address)
{
	// 포인터가 nullptr인데 접근하려 하면 크래시남. 그러므로 예외처리 ㅋ
	if (address == nullptr)
		return;

	*address = 5000;
}

void Calculate(int array[25])
{
	// 매개변수에 배열 이름을 넣는다는 건, 그 배열의 시작 메모리 주소를 전달한다는 것이다.
	// 따라서 함수 내부에서 배열 원소를 바꾸면 원본의 배열 데이터가 바뀐다.

	// 일반 변수는 복사되어 전달되므로 원본에는 영향이 없다.

	for (int i = 0; i < 25; i++)
	{
		array[i] = i + 1;
	}
}
```





## 포인터 연산

- 포인터 연산
  - +, - 2가지가 지원된다.  
  - 1을 더하면, 포인터 변수 타입의 메모리 크기만큼을 더해주게 된다.  
  - 2를 더하면 2 * 타입크기 만큼을 주소에 더해주게 된다.  

> 👉 역참조도 연산자로 취급되어서, 괄호로 묶지 않으면 우선순위에 따라 다르게 연산될 수 있음!  

```cpp
int array[100] = {};
int* arrayAddress = &array;

printf("array = %p\n", array);						    // array 변수의 시작 메모리 주소 출력
printf("&array[0] = %p\n", &array[0]);				// array 변수의 시작 메모리 주소 출력 (위와 동일)
printf("arrayAddress = %p\n", arrayAddress);				// array 변수의 시작 메모리 주소 출력 (위와 동일)
printf("arrayAddress + 1 = %p\n", arrayAddress + 1);		// 시작 메모리 주소에 + 4가 된 주소값이 나옴.

printf("*(arrayAddress + 3) = %d\n", *(arrayAddress + 3));	// array[3]의 값 출력.
printf("array[3] = %d\n", array[3]);				// array[3]의 값 출력.

// 역참조가 먼저 연산되어서, array[0]의 값에 +3이 되어 출력됨.
printf("*arrayAddress + 3 = %d\n", *arrayAddress + 3);		

*arrayAddress++ = 500;										// 역참조가 먼저 연산되어서 [0]에 500이 대입됨
*++arrayAddress = 777;										// array[1]에 777이 대입됨
```





## const 키워드와 포인터

포인터 변수도 const 키워드를 이용하여 상수로 만들 수 있다.  

- const int* addr;
  참조하는 메모리 주소는 변경할 수 있지만, '역참조 하여 그 값을 바꾸는' 건 불가능. (갖다 쓰는 건 가능)  
- int* const addr;
  처음 선언과 동시에 메모리 주소의 대입만 가능하고, 그 이후엔 다른 메모리의 주소를 저장할 수 없다.  
- const int* const addr;
  참조하는 대상도 변경할 수 없고, 참조하는 대상의 값도 변경할 수 없다. (주로 함수 인자 처리할 때 사용)  


```cpp
const int* cAddr = &number;
int number2 = 300;
cAddr = &number2;                       // 참조하는 메모리의 주소 변경은 가능
// *cAddr = 600;											  // lvalue 에러 (left value 에러. 대입되는 애들).
// 상수이므로 참조하는 메모리의 변수 값은 변할 수 없는데 변하게 해서 에러


int* const addrc = &number;
//addrc = &number2;											// lvalue 에러.
// 선언과 동시에 메모리 주소 대입만 가능하고, 그 이후엔 다른 메모리 주소 저장 불가능
```





## 구조체와 포인터

구조체도 포인터 변수 생성 할 수 있다.  
단 구조체의 멤버에 접근할 때 쓰는 '.'도 연산자로 취급되기 때문에,  
'*'와 '.'를 쓸 땐 괄호로 묶어야 한다.  
또는 '->' 를 사용하여 접근할 수 있다.  

```cpp
struct FPlayerInfo
{
	char Name[32];
	int Attack;
};

void main()
{
  FPlayerInfo info;
  FPlayerInfo* infoAddr = &info;

  info.Attack = 500;

  // *infoAddr.Attack = 600;									// 에러. *도 연산자, .도 연산자라서 처리가 불명확하다.
  (*infoAddr).Attack = 600;

  // '->' 는 포인터 변수가 가지고 있는 주소에 접근하여
  // 멤버변수를 사용할 수 있게 해준다.
  infoAddr->Attack = 600;
}
```



## 다중 포인터

다중포인터 : <u>이전 포인터 타입 변수의 메모리 주소</u>를 저장하는 기능이다.  
2중은 일반포인터 변수의 메모리 주소, 3중은 2중 포인터 변수의 메모리 주소를 저장하는 방식이다.  
포인터의 포인터.  


```cpp
int	Number = 100;
int* Addr = &Number;
int** Addr2 = &Addr;

printf("Number = %d\n", Number);              // 100 출력
printf("Number Address = %p\n", &Number);     // Number의 주소 출력

printf("Addr = %p\n", Addr);                  // Number의 주소 출력
printf("*Addr = %d\n", *Addr);                // Number의 값 100 출력
printf("Addr Address = %p\n", &Addr);         // Addr의 주소 출력

printf("Addr2 = %p\n", Addr2);                // Addr의 주소 출력

printf("*Addr2 = %p\n", *Addr2);              // Number의 주소 출력
                                              // Addr이 갖고 있는 건 Number의 주소이고
                                              // Addr2는 Addr의 주소를 갖고있고, 그걸 역참조 한거니까

printf("**Addr2 = %d\n", **Addr2);            // Number의 값 100 출력
printf("Addr2 Address = %p\n", &Addr2);       // Addr2의 주소 출력
```


### 이중포인터와 배열


```cpp
// 2차원 배열의 뒷자리 수 10(row, 세로칸 수)을 맞춰줘야 사용 가능
void ArrTest(int(*Arr)[10])
{

}

void main()
{
  int	Array[100] = {};
  int* ArrayAddr = Array;

  int* ArrayAddrArr[20] = {};

  int	Array1[5][10] = {};
  int** ArrayAddr1 = nullptr;
  //ArrayAddr1 = Array1;      // -> 에러. Array1는 2차원 배열일 뿐,
                              // 단일 포인터로 받을 수 있고 이중 포인터로 받을 수 없음.

  //int* ArrayAddr1[10] = {};
  //ArrayAddr1 = Array1;      // -> 에러. Array1은 2차원 배열이고, ArrayAddr1는 1차원 배열이니까

  int* ArrayAddr2 = &Array1[0][0];      // 2차원 배열의 한 칸의 주소를 가져와서 사용하는 것도 가능.

  ArrTest(Array1);
}
```



# 레퍼런스 (&)

다른 변수를 참조하여 값을 가져와서 사용하거나 값을 변경할 수 있게 해주는 기능.  
레퍼런스는 레퍼런스 변수를 선언하는 동시에 반드시 참조하는 대상을 지정해야 한다.  

`변수타입& 변수명 = 참조할 대상변수;` 의 방식으로 사용한다.  