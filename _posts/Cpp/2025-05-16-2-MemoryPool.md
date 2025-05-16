---
title:  "C++ 기초 정리 (메모리풀, 공유 메모리풀))"

categories:
  - Cpp
tags:
  - [Cpp, C++, MemoryPool]

toc: true
toc_sticky: true
 
date: 2025-05-16
last_modified_at: 2025-05-16
---

<br>

# MemoryPool 작성


```cpp
#include <iostream>
#include <stack>
#include <assert.h>

// 비타입 템플릿 인자 : 가변타입이 아닌 고정된 타입으로 템플릿 인자를 만들어주는 방식이다.
// 디폴트 인자도 되고 (int PoolSize = 1024), 상수로 인식되어 이 클래스 안에서 사용할 수 있다.
template <typename T, int PoolSize = 1024>
class CMemoryPool
{
public:
	CMemoryPool()
	{
		// ::operator new 를 하면 전역연산자 new를 호출한다.
		// 생성자 호출 없이 순수하게 메모리만 할당하는 방식이다.
		mMemory = (T*)(::operator new(PoolSize * sizeof(T)));

		//mMemory = (T*)new(PoolSize * sizeof(T));					// 원래 기본적으로 메모리 할당할 때 쓰는 new 를 사용하면, 생성자를 호출한다.

		//mMemory = (T*)malloc(PoolSize * sizeof(T));				// C언어 스타일의 메모리 생성방식. 이 경우에도 생성자 호출을 하지 않는다.

		// 스택에 인덱스 시작 주소를 담아둔다.
		for (int i = 0; i < PoolSize; ++i)
		{
			// 포인터 연산으로 메모리 주소 찾아갈 수 있도록.
			mAllocMemory.push(mMemory + i);
		}
	}

	~CMemoryPool()
	{
		::operator delete(mMemory);									// 위에서 전역 new 를 사용했기 때문에, 소멸도 전역 delete를 사용한다.

		// delete mMemory;											// 기본적인 delete 방식. 이 방식은 소멸자가 호출된다.

		// free(mMemory);											// C스타일의 동적 할당 해제. 소멸자 호출 X
	}

public:
	T* Alloc()
	{
		// 더이상 할당할 수 있는 공간이 없을 경우 예외처리를 해준다.
		assert(mAllocMemory.empty() == false && "Memory Pool Is Empty");			// 비어있으면 assert 발생

		//// 공간을 더 늘려서 재생성 해주어도 되지만, 기존의 할당되었던 메모리들을 다 제거하고 새로운 공간에 다시 들어가는 연산이 필요하므로 잘 사용하지 않음.

		// top을 이용해서 마지막에 추가된 데이터를 얻어온다.
		T* Ptr = mAllocMemory.top();

		// 마지막에 추가된 데이터를 제거한다.
		mAllocMemory.pop();

		// placement new
		// 1. 메모리 할당
		// 2. 객체 생성자 호출
		// 이미 할당된 주소에 객체를 생성할 때 사용한다.
		// 새로 메모리를 할당하는 방식이 아니다. 이미 할당된 주소에!
		//::new (Ptr) T;							// 전역 new. 생성자가 호출 됨. 

		return Ptr;
	}

	void DeAlloc(T* Ptr)
	{
		// 메모리를 풀에 반환할 때 소멸자를 호출하고 싶을 경우
		//Ptr->~T();

		// 삭제한다는 건, 다시 메모리 풀에 메모리를 반환한다는 것. 그러므로 풀에 푸시.
		mAllocMemory.push(Ptr);
	}

private:
	T* mMemory = nullptr;
	std::stack<T*>	mAllocMemory;					// Alloc Memory : '할당 가능한 메모리' 라는 뜻.
};

class CMonster
{
public:
	CMonster()
	{
		printf("CMonster 생성자\n");
	}

	~CMonster()
	{
		printf("CMonster 소멸자\n");
	}

public:
	char	mName[32] = {};

private:
	static CMemoryPool<CMonster>	mMonsterPool;

public:
	static void* operator new (size_t Size)
	{
		return mMonsterPool.Alloc();
	}

	static void operator delete (void* Ptr)
	{
		mMonsterPool.DeAlloc((CMonster*)Ptr);
	}
};

CMemoryPool<CMonster> CMonster::mMonsterPool;

int main()
{
	//CMemoryPool<CMonster>	MonsterPool;

	//CMonster* Monster1 = MonsterPool.Alloc();

	//MonsterPool.DeAlloc(Monster1);


	CMonster* Monster2 = new CMonster;						// operator new로 들어감.
	// 기본적인 new 의 형태이기 때문에 생성자가 알아서 호출되므로,
	// CMemoryPool의 생성자 코드에서 강제로 ::new (ptr) T; 를 통해 해당 타입의 생성자를 호출해주지 않아도 됨.
	// 이렇게 오퍼레이터를 오버라이드 해서 만들었기 때문에, CMemoryPool의 placement new 지워도 된다.


	delete Monster2;										// operator delete로 들어가서, 최종 delAlloc 호출 됨.
	// 마찬가지로 오퍼레이터 delete 오버라이드 했기 때문에, 소멸자 호출 됨.
	// 그래서 이런식으로 만든다면, 마찬가지로 CMemoryPool의 ptr->~T(); 지워도 됨.

	return 0;
}

```


# SharedMemoryPool

```cpp

#include <iostream>
#include <stack>
#include <assert.h>
#include <crtdbg.h>								// 메모리 누수 확인.

template <typename T, int PoolSize = 1024>
class CMemoryPool
{
public:
	CMemoryPool()
	{
		mMemory = (T*)(::operator new(PoolSize * sizeof(T)));

		// 스택에 인덱스 시작 주소를 담아둔다.
		for (int i = 0; i < PoolSize; ++i)
		{
			mAllocMemory.push(mMemory + i);
		}
	}

	~CMemoryPool()
	{
		::operator delete(mMemory);
	}

public:
	T* Alloc()
	{
		assert(!mAllocMemory.empty() && "Memory Pool Is Empty");

		T* Ptr = mAllocMemory.top();

		mAllocMemory.pop();

		return Ptr;
	}

	void DeAlloc(T* Ptr)
	{
		mAllocMemory.push(Ptr);
	}

private:
	T* mMemory = nullptr;
	std::stack<T*>	mAllocMemory;
};


// 공유하는 메모리풀 클래스.
// 특정 타입이 아니라, 1바이트 단위(unsigned char)로 메모리를 관리하도록 함.
// 이것도 내부 단편화가 생길 수 있지만,
// 풀을 사용하는 타입이 굉장히 많다면, 이걸 쓰는게 풀 관리가 더 쉬울 것.
template <int BlockSize, int BlockCount>
class CShareMemoryPool
{
public:
	CShareMemoryPool()
	{
		mMemory = (unsigned char*)(::operator new(BlockSize * BlockCount));

		for (int i = 0; i < BlockCount; ++i)
		{
			mAllocMemory.push(mMemory + (i * BlockSize));
		}
	}

	~CShareMemoryPool()
	{
		::operator delete(mMemory);
	}

public:
	// Alloc 함수 자체에 템플릿을 걸어서, Alloc 할 때 마다 '이 타입으로 만들고 싶어'라고 알린다.
	template <typename T>
	T* Alloc()
	{
		assert(sizeof(T) <= BlockSize);

		assert(!mAllocMemory.empty() && "Memory Pool Is Empty");

		T* Ptr = (T*)mAllocMemory.top();

		mAllocMemory.pop();

		return Ptr;
	}

	// 기존 MemoryPool 때와 달리, 어떤 타입이 들어올지 모르니 void* 로 인자 타입 변경.
	void DeAlloc(void* Ptr)
	{
		mAllocMemory.push((unsigned char*)Ptr);
	}

private:
	unsigned char* mMemory = nullptr;
	std::stack<unsigned char*>	mAllocMemory;
};

class CMonster
{
public:
	CMonster()
	{
		printf("CMonster 생성자\n");
	}

	virtual ~CMonster()
	{
		printf("CMonster 소멸자\n");
	}

protected:
	static int		mMaxSize;

public:
	// 풀을 사용할 타입마다 요구하는 최대사이즈가 다르다면, 이걸 통해 변경해줌.
	template <typename T>
	static void ComputeMaxSize()
	{
		size_t	Size = sizeof(T);
		if (Size > mMaxSize)
			mMaxSize = (int)Size;
	}

protected:
	static CShareMemoryPool<16, 200>	mShareMemoryPool;
};

class CGoblin :
	public CMonster
{
public:
	CGoblin()
	{
		printf("CGoblin 생성자\n");
	}

	virtual ~CGoblin()
	{
		printf("CGoblin 소멸자\n");
	}

private:
	static CMemoryPool<CGoblin>	mGoblinPool;

public:
	static void* operator new (size_t Size)
	{
		return mShareMemoryPool.Alloc<CGoblin>();
	}

	static void operator delete (void* Ptr)
	{
		mShareMemoryPool.DeAlloc(Ptr);
	}
};

class COrc :
	public CMonster
{
public:
	COrc()
	{
		printf("COrc 생성자\n");
	}

	virtual ~COrc()
	{
		printf("COrc 소멸자\n");
	}

public:
	int		mNumber = 100;

private:
	static CMemoryPool<COrc>	mOrcPool;

public:
	static void* operator new (size_t Size)
	{
		return mShareMemoryPool.Alloc<COrc>();
	}

	static void operator delete (void* Ptr)
	{
		mShareMemoryPool.DeAlloc(Ptr);
	}
};

CMemoryPool<CGoblin> CGoblin::mGoblinPool;
CMemoryPool<COrc> COrc::mOrcPool;
CShareMemoryPool<16, 200> CMonster::mShareMemoryPool;

int CMonster::mMaxSize = 0;


int main()
{
	// 메모리 누수 탐지. 누수가 탐지되면 출력 창에 DETECTED MEMORY LEAK 이라는 로그를 출력함.
	_CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);
	// 로그에서 출력된 [블록번호] 를 인자로 넘겨주면, 중단점이 잡힘.
	_CrtSetBreakAlloc(1309);

		
	CMonster::ComputeMaxSize<CGoblin>();			// CGoblin의 최대 사이즈 변경.
	CMonster::ComputeMaxSize<COrc>();				// 마찬가지로 최대 사이즈 변경.
	

	printf("Goblin Size : %d\n", sizeof(CGoblin));					// 8바이트 출력.
	printf("Orc Size : %d\n", sizeof(COrc));						// 16바이트 출력.
	// virtual 소멸자 때문에 8바이트 + int mNumber로 인해 4바이트 추가되어 12바이트지만,
	// 데이터 패딩으로 인해 16바이트로 늘어남.


	// 기존 방식대로 각각의 타입별 pool을 만드는게 좋긴 하지만,
	// 통합 오브젝트 풀이 필요할 수 있음. (타입별 pool을 많이 만들면, 이것 역시 내부 단편화가 많이 생길 수 있음)
	// 통합 오브젝트 풀 관리 하는 법
	// 1. 통합 오브젝트 풀을 만들어서, 균등한 블럭으로 나눈다.
	// 일부 블럭은 고블린, 일부 블럭은 오크 이런식으로.
	// 2. 요청할 때 마다 가변적인 공간을 제공하여 제공. 고블린은 10 요구하면 10 주고, 오크는 3 필요하면 3 주고.
	// 3. 혹은 부모 클래스에 static으로 maxSIze 변수를 만든 뒤, 상속받는 자식 클래스들의 size를 구하여 size만큼 미리 pool을 만들어 놓는다거나.

	CMonster* Monster[10] = {};

	Monster[0] = new CGoblin;
	Monster[1] = new COrc;


	delete Monster[0];
	delete Monster[1];

	return 0;
}

```