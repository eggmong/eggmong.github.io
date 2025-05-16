---
title:  "C++ 기초 정리 (shared_ptr 직접 작성, 변환연산자 작동 원리)"

categories:
  - Cpp
tags:
  - [Cpp, C++, shared_ptr]

toc: true
toc_sticky: true
 
date: 2025-05-16
last_modified_at: 2025-05-16
---

<br>

변환연산자를 통해 어떻게 int형 변수에 포인터를 넣을 수 있는지?

```cpp

#include <iostream>
#include <stack>
#include <assert.h>
#include <crtdbg.h>


template <typename T>
class TSharedPtr
{
	class CRef
	{
	public:
		CRef()
		{
		}

		~CRef()
		{
			if (mPtr)
				delete mPtr;
		}

	public:
		T* mPtr = nullptr;
		int	mRefCount = 0;
	};

public:
	TSharedPtr()
	{
		mRef = new CRef;
		mRef->mPtr = new T;
		++mRef->mRefCount;
	}

	// 저장을 하기 위한 동적할당. 인자로 포인터를 넣는다.
	TSharedPtr(const T& Ptr)
	{
		// T포인터를 직접 넣어준다는 건, 포인터가 새로 공간이 만들어진다는 것.
		mRef = new CRef;
		++mRef->mRefCount;

		mRef->mPtr = new T;
		*mRef->mPtr = Ptr;
	}

	TSharedPtr(T* Ptr)
	{
		mRef = new CRef;
		++mRef->mRefCount;

		mRef->mPtr = Ptr;
	}

	TSharedPtr(const TSharedPtr<T>& Ptr)
	{
		// 복사 생성자.

		mRef = Ptr.mRef;

		if (mRef)
			++mRef->mRefCount;								// 참조가 일어나므로 참조 카운트 증가.
	}

	TSharedPtr(const TSharedPtr<T>&& Ptr)
	{
		// 이동 연산자.

		mRef = Ptr.mRef;

		Ptr.mRef = nullptr;
	}

	~TSharedPtr()
	{
		if (mRef)
		{
			--mRef->mRefCount;

			if (mRef->mRefCount == 0)
			{
				delete mRef;
			}
		}
	}

public:
	void operator = (const TSharedPtr<T>& Ptr)
	{
		mRef = Ptr.mRef;

		if (mRef)
			++mRef->mRefCount;
	}

	void operator = (T* Ptr)
	{
		if (mRef)
		{
			delete mRef->mPtr;
			delete mRef;
		}

		mRef = new CRef;
		++mRef->mRefCount;

		mRef->mPtr = Ptr;
	}

	void operator = (T Ptr)
	{
		if (mRef)
		{
			delete mRef->mPtr;
			delete mRef;
		}

		mRef = new CRef;

		mRef->mPtr = new T;

		*mRef->mPtr = Ptr;
		++mRef->mRefCount;
	}

	T* operator -> ()
	{
		return mRef->mPtr;
	}

	const T* operator -> ()	const
	{
		return mRef->mPtr;
	}

	T& operator * ()
	{
		return *(mRef->mPtr);
	}

	const T& operator * ()	const
	{
		return *(mRef->mPtr);
	}

	// 변환 연산자 작성.
	operator const T& ()	const
	{
		return *(mRef->mPtr);
	}

private:
	CRef* mRef = nullptr;

public:
	// 동적 할당해서 shared_ptr 만드는 함수 작성함.
	static const TSharedPtr<T>& make_shared(const T& Data)
	{
		return TSharedPtr<T>(Data);
	}
};

class CTest
{
public:
	CTest()
	{
	}

	~CTest()
	{
	}

public:
	int	mNum = 100;
};

int main()
{
	_CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);

	int* Number1 = new int;

	TSharedPtr<int>	IntPtr;
	TSharedPtr<int>	IntPtr1 = IntPtr;
	TSharedPtr<int>	IntPtr2(Number1);

	*IntPtr = 500;

	int	Number = IntPtr;								// 변환 연산자로 인해 포인터를 int 타입 변수에 넣는 것도 가능해짐.

	*IntPtr2 = 300;

	printf("%d\n", *IntPtr);
	printf("%d\n", *IntPtr1);
	printf("%d\n", *IntPtr2);




	CTest* Test3 = new CTest;
	TSharedPtr<CTest>	Test1(Test3);
	TSharedPtr<CTest>	Test2 = Test1;

	Test1->mNum = 500;
	Test2->mNum = 800;

	printf("%d\n", Test3->mNum);					// Test1, Test2 모두 Test3을 가리키고 있음. 최종적으로 800 출력.

	return 0;
}
```