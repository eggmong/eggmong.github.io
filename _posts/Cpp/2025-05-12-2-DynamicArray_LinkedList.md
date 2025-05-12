# 동적 배열


> 템플릿으로 함수나 클래스를 작성할 경우, 헤더와 cpp를 분리해서 선언/구현 따로 두면,  
> cpp에서 템플릿 사용할 때 어떤 타입에 대해 인스턴스화할지 알 수 없기 때문에 링크 에러 발생.  


```cpp
// Array.h

#pragma once

#include <assert.h>

template <typename T>
class CArray
{
public:
	CArray()
	{
		mArray = new T[mCapacity];
		memset(mArray, 0, sizeof(T) * mCapacity);
	}

	~CArray()
	{
		delete(mArray);
		mArray = nullptr;
	}

private:
	T* mArray = nullptr;
	int mSize = 0;
	int mCapacity = 2;

public:
	//void push_back(T data)				
	//{
	//	// 문제점: 에러는 아니지만. 최종 복사가 2번 일어나게 됨. 구조체 크기가 클 수록 치명적.
	//	// 함수 인자 복사, 컨테이너 데이터를 저장할 때 발생하는 복사
	// 
	//	// 복사 줄이는 법: 참조를 하거나, 포인터로 받는다.
	//}

	//void push_back(T& data)
	//{
	//	// 인자에 &를 붙여서 레퍼런스로 받아 참조형태로 바꿔 복사가 줄여짐.
	//	// 하지만 const 를 붙이지 않았으므로, 인자로 들어오는 값을 내부에서 변경할 수 있음.
	//}

	// 최종형태. 값이 아닌 참조로 받고, const를 붙여 변경할 수 없도록 함.
	// 새로운 요소를 끝에 삽입
	void Push_Back(const T& data)
	{
		// 배열이 꽉 찼는지 확인
		if (mSize >= mCapacity)
		{
			mCapacity *= 2;

			T* array = new T[mCapacity];

			// 새로운 공간에 기존 배열의 내용을 복사.
			memcpy(array, mArray, sizeof(T) * mSize);

			// 기존 배열 제거
			delete[] mArray;

			// 새로운 배열의 주소를 멤버 포인터에 넣어준다.
			mArray = array;
		}

		mArray[mSize] = data;
		mSize++;
	}

	// 마지막에 추가된 데이터를 제거
	void Pop_Back()
	{
		// assert: 디버그 모드일때만 동작한다. 인자로 조건을 주고, 거짓일 경우 프로그램을 강제로 종료한다.
		assert(mSize > 0);

		mSize--;	// 어차피 배열이라 요소 하나만 지우고 그런거 못 함.
		// 그냥 인덱스를 감소시켜서, 나중에 추가할 때 덮어씌우도록 함.
	}

	int GetSize() const
	{
		return mSize;
	}

	int GetCapacity() const
	{
		return mCapacity;
	}

	bool IsEmpty() const
	{
		return mSize <= 0;
	}

	void Clear()
	{
		mSize = 0;
	}

	void EraseIndex(int index)
	{
		// 인자에 해당되는 인덱스를 찾아 지움
		// 즉, 제거한 후 뒤의 값들을 1칸씩 앞으로 이동
		
		for (int i = index; i < mSize - 1; i++)
		{
			mArray[i] = mArray[i + 1];
		}

		mSize--;
	}

	void EraseIndex1(int index)
	{
		mArray[index] = mArray[mSize - 1];
		mSize--;

		// 마지막에 추가된 데이터를 지우려는 인덱스에 넣어주고
		// 추가된 수를 줄여버린다.

		// 속도는 빠르지만, 정렬이 안 됨.
	}

	void EraseValue(const T& data)
	{
		// 인자로 들어온 값을 찾아서 해당 인덱스를 지움

		for (int i = 0; i < mSize; i++)
		{
			if (mArray[i] == data)
			{
				EraseIndex(i);
				break;
			}
		}
	}

	T& operator [] (int index) const
	{
		// 인자의 결과가 거짓일 경우 assert가 동작함, 그러니 참인 조건을 넣어줘야 함.
		assert(index >= 0 && index < mSize);

		return mArray[index];
	}
};
```


```cpp
// main.cpp

#include <iostream>
#include "CArray.h"

// 템플릿으로 구현한 가변 배열
// 템플릿으로 함수나 클래스를 작성할 경우, 헤더를 분리하면 cpp가 링크를 못 함.
// 그래서 헤더에다 선언과 구현까지 다 해야 함. = inline

struct FTest
{
	char Arr[1024];
};

int main()
{
	CArray<FTest> arr;                         
	
	FTest test;
	arr.Push_Back(test);				// 문제점: 에러는 아니지만. 최종 복사가 2번 일어나게 됨.
										// 구조체 크기가 클 수록 치명적.
										// 복사 줄이는 법: 참조를 하거나, 포인터로 받는다.

	CArray<int> arr1;
	for (int i = 0; i < 10; i++)
	{
		arr1.Push_Back(i + 1);
	}

	int size = arr1.GetSize();

	arr1.EraseIndex(7);

	return 0;
}
```




# 이중 연결 리스트


## friend 키워드

friend로 선언된 클래스는 이 클래스의 private 접근 가능.  
CLinkedList 클래스는 CListNode의 private에 접근이 가능 해졌다.  


```cpp
// LinkedList.h

#pragma once

#include <assert.h>

template <typename T>
class CListNode
{
	// friend로 선언된 클래스는 이 클래스의 private 접근 가능.
	// CLinkedList 클래스는 CListNode의 private에 접근이 가능 해졌다.

	template <typename T>	// 템플릿 클래스에서 friend를 사용할 경우, friend class에도 템플릿 선언 필요.
	friend class CLinkedList;

private:
	CListNode()			// 생성자를 private 으로 두어, 외부에서 생성하지 못하도록 함.
	{
		
	}

	~CListNode()
	{
		
	}

private:
	T mData;
	CListNode<T>* mNext = nullptr;
	CListNode<T>* mPrev = nullptr;
};

template <typename T>
class CLinkedList
{
public:
	CLinkedList()
	{
		mBegin = new CListNode<T>;
		mEnd = new CListNode<T>;

		mBegin->mNext = mEnd;
		mEnd->mPrev = mBegin;
	}

	~CLinkedList()
	{
		CListNode<T>* current = mBegin;
		while (current)
		{
			CListNode<T>* next = current->mNext;
			delete current;
			current = next;
		}
	}

	void push_back(const T& data)
	{
		CListNode<T>* node = new CListNode<T>;
		node->mData = data;

		CListNode<T>* prev = mEnd->mPrev;

		prev->mNext = node;
		node->mPrev = prev;

		node->mNext = mEnd;
		mEnd->mPrev = node;

		mSize++;

		// 똑같이 push_back 할 때 array가 빠른지, linkedList가 빠른지?
		// 배열이 꽉 차서 늘려야 할 땐 array가 느림.
		// 다만 평소엔 array가 연속된 메모리 공간에 있으므로 빠름.
		// 늘리는 건 linkedList가 빠름. 그냥 하나 추가하면 되니까.
	}

	void push_front(const T& data)
	{
		// begin과 begin 다음 노드에 추가한다. 

		CListNode<T>* node = new CListNode<T>;
		node->mData = data;

		CListNode<T>* next = mBegin->mNext;

		node->mPrev = mBegin;
		mBegin->mNext = node;

		node->mNext = next;
		next->mPrev = node;

		mSize++;
	}

	void pop_back()
	{
		CListNode<T>* deleteNode = mEnd->mPrev;

		CListNode<T>* deletePrevNode = deleteNode->mPrev;

		mEnd->mPrev = deletePrevNode;
		deletePrevNode->mNext = mEnd;

		delete deleteNode;

		mSize--;
	}

	void pop_front()
	{
		// 지울 노드
		CListNode<T>* deleteNode = mBegin->mNext;

		// 지울 노드의 다음 노드
		CListNode<T>* deleteNextNode = deleteNode->mNext;

		mBegin->mNext = deleteNextNode;
		deleteNextNode->mPrev = mBegin;

		delete deleteNode;

		mSize--;
	}


private:
	CListNode<T>* mBegin;
	CListNode<T>* mEnd;

	int mSize = 0;
};
```