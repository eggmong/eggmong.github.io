---
title:  "C++ 기초 정리 (스마트포인터)"

categories:
  - Cpp
tags:
  - [Cpp, C++, shared_ptr, weak_ptr, unique_ptr]

toc: true
toc_sticky: true
 
date: 2025-05-16
last_modified_at: 2025-05-16
---

<br>

```cpp

#include <iostream>

// 동적할당을 사용할 때 생길 수 있는 문제점.
// 1. 메모리 단편화
//// 외부 단편화: 메모리에 빈 공간이 있긴 하지만 짜투리 공간만 남아서, 큰 메모리 할당을 할 수 없을 때)
//// 내부 단편화: 할당된 메모리 중에서 실제로 사용되지 않는 부분이 낭비되는 상황. (구조체나 클래스에서 메모리 패딩 상황.)
//// 해결 방법
//// - 메모리 풀 (오브젝트 풀링) : 메모리 미리 여러개 할당 해놓고, 돌려가며 쓰기. (내부, 외부 단편화 모두 해결)
//// - 커스텀 할당자 : 
//// - 페이징 기법 : 가상 메모리를 같은 크기의 블럭으로 나눈 것을 페이지라고 하는데, 메모리를 페이지와 같은 크기로 나눈 것을 프레임이라고 한다.
////				 사용하지 않는 프레임을 페이지에 옮기고, 필요한 메모리를 페이지 단위로 프레임에 옮겨주는 기법. (외부 단편화 해결 방법)
//// - 세그멘테이션 기법 : 가상 메모리를 서로 다른 크기로 분할해서 메모리를 할당하여, 실제 메모리 주소로 변환하여 사용하는 기법. (내부 단편화 해결 방법)
// 2. 메모리 릭
//	  메모리 해제가 안되는 것.
// 3. 댕글링 포인터의 잘못된 접근
//	  메모리가 해제 되었는데 포인터가 접근하는 것.
////	메모리 릭, 댕글링 포인터 해결 방법 : 스마트 포인터 사용.
////	- 스마트 포인터: 레퍼런스 카운터를 사용하여 참조하는지 확인하고, 참조 카운트가 없으면 메모리를 제거.

// c++에서 제공하는 스마트 포인터는 std namespace 안에 들어 있다.
// c++에서 제공하는 스마트 포인터 종류
// std::unique_ptr : 단 하나의 스마트 포인터만 소유 가능. 복사 안됨. 속도가 빠름.
// std::shared_ptr : 가장 많이 씀. 여러 객체가 하나의 공간을 공유하는 경우. 레퍼런스 카운트를 이용하여 참조하는 횟수를 만들고, 레퍼런스 카운트가 0이 되면 메모리를 제거하는 방식. 속도가 느림.
// std::weak_ptr : shared_ptr의 비소유 참조. 레퍼런스 카운트를 증감시키지 않음. 속도가 빠르다.


int main()
{
	// std::unique_ptr
	// std::make_unique : unique_ptr을 생성해주는 함수
	std::unique_ptr<int> uniquePtr = std::make_unique<int>(10);

	printf("unique_ptr : %d\n", *uniquePtr);				// 10 출력.
	
	//std::unique_ptr<int> copy = ptr;				// 에러. 복사 안 됨.
	std::unique_ptr<int> move = std::move(uniquePtr);		// 이동은 됨.

	printf("unique_ptr : %d\n", *uniquePtr);				// 에러. 위에서 move로 소유권을 이전시켰으므로, ptr은 nullptr을 가리키게 되었으므로.

	// ------------------------

	// std::shared_ptr

	int* number = new int;
	*number = 500;
	std::shared_ptr<int> sharedPtr(number);					// 레퍼런스 카운트 1 됨.
	// 이렇게 shared_ptr로 만든 후엔 number를 사용하지 않고 shared_ptr을 사용하는 게 좋다.
    std::cout << sharedPtr.use_count();						// 1 출력
	printf("%d\n", *sharedPtr);								// 500 출력.

	std::shared_ptr<int> sharedPtr1 = std::make_shared<int>(123);		// 이렇게도 가능.

	std::shared_ptr<int> sharedPtr2 = sharedPtr;				// 대입받아서도 사용 가능. 레퍼런스 카운트 2가 됨.

	printf("%d\n", *sharedPtr1);							// 123 출력
	printf("%d\n", *sharedPtr2);							// 500 출력

	// ----------------

	// weak_ptr

	// 참조카운트가 증가하지 않는다.
	std::weak_ptr<int> wPtr = sharedPtr;

	// 그러므로 유효성 확인 후 사용해야 한다.
	if (wPtr.expired())						
	{
		// printf("%d\n", *wPtr);			// 에러. 역참조 기능 없음.
		std::shared_ptr<int> sharedLock = wPtr.lock();		// shared_ptr에 lock을 걸어 가져온다.
	}


	return 0;
}
```