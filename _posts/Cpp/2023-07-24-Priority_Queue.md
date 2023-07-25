---
title:  "[STL] 우선순위 큐 (priority_queue)"

categories:
  - Cpp
tags:
  - [Cpp, C++, STL]

toc: true
toc_sticky: true
 
date: 2023-07-24
last_modified_at: 2023-07-24
---

<br>

## Priority_Queue (우선순위 큐) 란?
- Queue의 한 종류로, <b>우선순위에 따라 정렬</b>된 Queue이다.  
- 어떤 원소가 삽입(push) 된다면 뒤에 삽입되었다가 지정된 우선순위에 맞춰서 Queue가 정렬된다.  
- 삭제(pop)는 정렬된 Queue의 맨 앞에서 이루어진다.  
- 자료구조 Heap으로 구현되었기 때문에 Heap의 시간복잡도와 같은 O(logN)이다.  
- 여기서 heap 과 다른 점이 나타나는데, <mark style='background-color: #fff5b1'>heap은 make_heap으로 만들 수 있고 이것은 vector 데이터를 heap 구조로 변경해주는 것이다.</mark>  
- 우선순위 큐는 내부적으로 make_heap, push_heap, pop_heap 을 사용하지만 <mark style='background-color: #fff5b1'>Priority_Queue 라는 Container를 제공해준다.</mark>  

## C++ STL에서의 Queue

먼저 queue 라이브러리를 include 해줘야 한다.  

```cpp
#include <queue>
```

그리고 이런식으로 선언할 수 있다.  

```cpp
priority_queue<자료형> 변수명  // 원소가 내림차순(기본, less)으로 정렬되는 우선순위 큐가 생성된다.  
priority_queue<int> queue;  

priority_queue<자료형, Container> 변수명  // 내부 컨테이터 변경
priority_queue<int, vector<int>> queue;  // 내부 컨테이너는 구조체를 넣어줄 수도 있다. 이땐 비교 연산자를 정의해줘야 한다.

priority_queue<자료형, Container, 정렬기준> 변수명  // 정렬 기준 변경
priority_queue<int, vector<int>, greater<int>> queue;  // 오름차순으로 정렬 기준 변경. 정렬 기준은  연산자 오버로딩으로 재정의할 수 있다.
```

## 멤버 함수

- push(element) : 원소 삽입 (정렬 기준에 따라 내부적으로 정렬 된다)  
- pop() : 맨 앞에 있는 원소 삭제  
- top() : 맨 앞에 있는 원소를 반환  
- empty() : 큐가 비어있으면 true, 아니면 false 반환  
- size() : 큐의 크기 반환  

## 예제

```cpp
#include <iostream>
#include <queue>
using namespace std;
 
struct Sample
{
    int index;
    int value;

    Sample(int _index, int _value)
    {
        index = _index;
        value = _value;
    }
};

struct Compair
{
    bool operator() (Sample s1, Sample s2)
    {
        // value가 큰 순서대로 정렬된다
        return s1.value < s2.value;
    }
};

void main()
{
    priority_queue<Sample, vector<Sample>, Compair> queue;

    queue.push(Sample(0, 111));
    queue.push(Sample(1, 222));
    queue.push(Sample(2, 333));
    queue.push(Sample(3, 444));
    queue.push(Sample(3, 555));

    Sample popValue = queue.top();  // { index=3 value=555 }
    queue.pop();
    popValue = queue.top();         // { index=3 value=444 }
    queue.pop();
}
```

이렇게 구조체를 넣어줄 수도 있고, 비교 기준을 바꿔줄 수도 있다.  
( + 만약 int가 아니라 char 자료형이라면, 문자의 아스키코드 값이 더 큰 순서대로 정렬될 것 )  

