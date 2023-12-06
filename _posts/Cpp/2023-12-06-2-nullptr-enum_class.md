---
title:  "새로운 키워드 (C++11~) / nullptr, enum class"

categories:
  - Cpp
tags:
  - [Cpp, C++, nullptr, enum class]

toc: true
toc_sticky: true
 
date: 2023-12-06
last_modified_at: 2023-12-06
---

<br>

> 🤯 언리얼을 하기 위해 C++ 기억 되살리기 프로젝트

# 새로운 키워드 (C++11~) / nullptr, enum class

- nullptr (널 포인터)
- 고정 폭 정수형
- enum class

## 1. nullptr

- `NULL` 을 쓰면 가끔 이상한 문제가 발생함

```cpp
// Class.h
float GetScore(const char* name);
float GetScore(int id);

// Main.cpp
Class* myClass = new Class("COMP3100");
// ...
int score = myClass->GetScore(NULL);            // <- 어떤 함수가 호출될까?
```

> char* 포인터로 받는 함수가 호출될 것이라고 생각할 수 있지만,
> int형을 받는 함수가 호출된다.

```cpp
int number = NULL;                              // OK
int* ptr = NULL;                                // OK

int anotherNumber = nullptr;                    // ERROR;
int* anotherPtr = nullptr;                      // OK
```

- NULL
  - `#define NULL 0`
  - 숫자임;
- nullptr
  - `typedef decltype(nullptr) nullptr_t;`
  - null 포인터 상수
  - 포인터에는 언제나 nullptr을 쓰자



## 2. 고정 폭 정수형

int가 4바이트라는 표준은 사실 없음.  
많이 쓰는 플랫폼에서 4바이트일 뿐이고, 보장은 없음.  
그래서 고정 폭 정수형이 등장함.  

- int8_t / uint8_t : 8비트 int 부호 있는거/없는거
- int16_t / uint16_t
- int32_t / uint32_t
- int64_t / uint64_t
- intptr_t / uintptr_t
- 등등..
  - 가독성 향상을 위해 낡은 기존 자료형보다 이것들을 활용하자~
  - `int8_t score = student->GetScore();`



## 3. enum class

- 문제점 : C스타일 enum
  ```cpp
  // Main.cpp
  enum eScoreType
  {
    Assignment1,
    Assignment2,
    Assignment3,
    Midterm,
    Count,
  };

  enum eStudyType
  {
    Fulltime,
    PartTime,
  };

  // Main.cpp
  int main()
  {
    eScoreType type = Midterm;
    eStudyType studyType = FullTime;

    int num = Assignment3;                              // 문제 발생!

    if (type == FullTime)                               // 문제 발생!
    {
        // ...
    }

    return 0;
  }
  ```

> 컴파일러가 타입 체크를 해주지 않는다는 단점이 있었음!

- 해결책 : enum class
  ```cpp
  // Main.cpp
  enum class eScoreType
  {
    Assignment1,
    Assignment2,
    Assignment3,
    Midterm,
    Count,
  };

  enum class eStudyType
  {
    Fulltime,
    PartTime,
  };

  // Main.cpp
  int main()
  {
    eScoreType type = eScoreType::Midterm;
    eStudyType studyType = eStudyType::FullTime;

    int num = eScoreType::Assignment3;                              // 에러 출력함

    if (score == eStudyType::FullTime)                              // 에러 출력함
    {
        // ...
    }

    return 0;
  }
  ```

> 컴파일 에러 출력을 해주므로 실수를 잡을 수 있다.

- 정수형으로의 암시적 캐스팅이 없음
- 자료형 검사(타입체크) 해준다
- 또한 enum에 할당할 바이트 양을 정할 수도 있음
  - 파일 저장할 때 유용...

```cpp
#include <cstdint>
enum class eScoreType : uint8_t         // 8비트 int에 집어넣겠다는 명시
{
    Assignment1,
    Assignment2,
    Assignment3,
    Midterm,
    Final = 0x100,          // 경고 발생
                            // 0x100은 256인데 8비트에 할당할 수 없어...
};
```

