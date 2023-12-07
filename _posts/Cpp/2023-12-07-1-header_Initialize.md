---
title:  "헤더에서 초기화하기"

categories:
  - Cpp
tags:
  - [Cpp, C++, header, initialize]

toc: true
toc_sticky: true
 
date: 2023-12-07
last_modified_at: 2023-12-07
---

<br>

> 🤯 언리얼을 하기 위해 C++ 기억 되살리기 프로젝트

# 헤더에서 초기화하기

## 사례 1.

```cpp
//Vector.h

class Vector
{
    // ...
    private:
        int mX = 0;                             // OK
        int mY = 0;                             // OK

        const static int mDimension = 2;        // OK
        static int mCount = 0;                  // 에러! 이건 아직 지원 안 됨.
};
```



## 사례 2.

```cpp
// Vector.h

class Vector
{
    // ...
    private:
        static int mCount;
};

// vector.cpp
// const static이 아닌 것 때문에
// 계속 이렇게 해야 함
int Vector::mCount = 0;
```

그렇지만 쓰지 않는 게 좋다.  
헤더 파일은 계속 어딘가에 include 될 수 있고,  
그렇게 되면 코드가 커지기 때문에  
헤더를 변경할 때 마다
- 헤더를 include하는 모든 .cpp 파일을 재빌드 해야 함
- 이 헤더를 다른 헤더들이 include 할 수 있고
- 그 헤더를 또 다른 헤더들을...
