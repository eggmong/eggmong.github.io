---
title:  "STL 알고리듬"

categories:
  - Cpp
tags:
  - [Cpp, C++, STL]

toc: true
toc_sticky: true
 
date: 2023-12-05
last_modified_at: 2023-12-05
---

<br>

> 🤯 언리얼을 하기 위해 C++ 기억 되살리기 프로젝트

# STL 알고리듬

- 요소 범위에서 쓸 수 있는 함수들
  - [처음, 마지막)
- 배열 또는 몇몇 STL 컨테이너에서 쓸 수 있음
- 반복자를 통해 컨테이너에 접근
- 컨테이너의 크기를 변경하지 않음 (따라서 추가 메모리 할당도 없음)

## 1. 알고리듬 유형

- #include <algorithm>
  - 변경 불가 순차 연산
    - find(), for_each(), ...
  - 변경 가능한 순차 연산
    - copy(), swap(), ...
  - 정렬 관련 연산
    - sort(), merge(), ...
- #include <numeric>
  - 범용 수치 연산
    - accumulate(), ...

```cpp
std::copy(scores.begin(), scores.end(), copiedScores.begin());`

// scores의 처음부터 끝까지 복사해달라,  
// 어느 위치냐면 copiedScores의 처음 위치에.  
```

### 1-1. find() 알고리즘

예시로 만들어본 find 알고리즘.  

```cpp
template <typename ITER, typename T>
const ITER* find(const ITER* begin, const ITER* end, const T& value)
{
    const ITER* p = begin;
    while (p != end)
    {
        if (*p == value)
        {
            break;
        }

        ++p;
    }

    return p;               // 위치, 즉 포인터를 반환한다
}
```

