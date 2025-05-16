```cpp
#include <iostream>
#include <stack>
#include <assert.h>

// 메모리 풀링

// 비타입 템플릿: 가변타입이 아닌, 고정된 타입으로 템플릿 인자를 만들어주는 방식이다.
// 디폴트 인자도 되고(= 1024), 상수로 인식되어 이 클래스 안에서 사용할 수 있다.
template <typename T, int PoolSize = 1024>
class CMemoryPool
{
	
public:
	CMemoryPool()
	{
		//mMemory = (T*)new(PoolSize * sizeof(T));					// 원래 기본적으로 메모리 할당할 때 쓰는 new 를 사용하면, 생성자를 호출한다.

		mMemory = (T*)(::operator new(PoolSize * sizeof(T)));		// 하지만 ::operator new 를 하면, 전역연산자 new를 호출한다.
																	// 생성자 호출 없이 순수하게 메모리만 할당하는 방식이다.

		// mMemory = (T*)malloc(PoolSize * sizeof(T));					// C언어 방식의 동적할당. 이 경우에도 생성자 호출을 하지 않는다.


		// 스택에 인덱스 시작 주소를 담아둔다.
		for (int i = 0; i < PoolSize; i++)
		{
			// 포인터 연산으로, 메모리사이즈
			mAllocMemory.push(mMemory + i);
		}
	}

	~CMemoryPool()
	{
		// delete mMemory;

		::operator delete(mMemory);									// 전역 new 를 사용했기 떄문에, 소멸도 전역 delete를 사용한다.

		// free(mMemory);												// C언어의 동적 할당 해제.
	}

	T* Alloc()
	{
		// 더이상 할당할 수 있는 공간이 없을 경우, 예외처리를 해도 되고 -> 이 방식이 나음.
		// 공간을 더 늘려서 재생성 해주어도 된다. -> 기존의 할당되었던 메모리들이 제거되고 새로운 공간에 다시 들어가는 연산이 필요하므로 힘듦.

		assert(mAllocMemory.empty() == false && "Memory pool is empty.");		// 비어있으면 assert 발생
		
		// top을 이용해서 마지막에 추가된 데이터를 얻어옴.
		T* ptr = mAllocMemory.top();
		mAllocMemory.pop();		// 마지막 데이터 제거

		// placement new
		// 1. 메모리 할당
		// 2. 객체 생성자 호출
		// 이미 할당된 주소에 객체를 생성할 때 사용한다.
		// 새로 메모리를 할당하는 게 아님! 이미 할당된 주소에! 
		::new (ptr) T;		// placement new. 생성자가 호출 됨. ::를 붙여 전역적인 new.

		return ptr;
	}

	void DeleteAlloc(T* ptr)
	{
		// 메모리를 풀에 다시 반환할 때, 소멸자를 호출하고 싶다면
		ptr->~T();

		// 삭제한다는 건, 다시 메모리 풀에 메모리를 반환한다는 것.
		mAllocMemory.push(ptr);
	}

private:
	T* mMemory = nullptr;

	std::stack<T*> mAllocMemory;			// Alloc Memory : 할당 가능한 메모리라는 뜻.
};

class CMonster
{

public:
	CMonster()
	{
		printf("CMonster 생성자 \n");
	}

	~CMonster()
	{
		printf("CMonster 소멸자 \n");
	}

	char mName[32] = {};

private:
	static CMemoryPool<CMonster> mMonsterPool;

public:
	static void* operator new (size_t size)
	{
		return mMonsterPool.Alloc();
	}

	static void operator delete (void* ptr)
	{
		mMonsterPool.DeleteAlloc((CMonster*)ptr);
	}
};

CMemoryPool<CMonster> CMonster::mMonsterPool;

int main()
{
	//CMemoryPool<CMonster> monsterPool;

	//CMonster* monster1 = monsterPool.Alloc();

	//monsterPool.DeleteAlloc(monster1);

	CMonster* monster2 = new CMonster;		// operator new로 들어감.
	// 기본적인 new 의 형태이기 떄문에 생성자가 호출되므로,
	// CMemoryPool의 생성자 코드에서 강제로 ::new (ptr) T; 코드를 통해 생성자 호출해주지 않아도 됨.
	// 이렇게 오퍼레이터를 오버라이드 해서 만들어서, ::new (ptr) T; 이 코드 지워도 된다.

	delete monster2;			// operator delete으로 들어가서 deleteAlloc 호출됨.
	// 마찬가지로 오퍼레이터 delete 오버라이드 했기 때문에, 소멸자 호출 됨.
	// 그래서 이런식으로 만든다면, 역시 CMemoryPool 클래스의 소멸자에서 ptr->~T(); 이 코드 지워도 됨.

	return 0;
}
```