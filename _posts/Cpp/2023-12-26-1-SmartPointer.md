---
title:  "스마트 포인터 / unique_ptr 유니크 포인터"

categories:
  - Cpp
tags:
  - [Cpp, C++, pointer, smart pointer]

toc: true
toc_sticky: true
 
date: 2023-12-26
last_modified_at: 2023-12-26
---

<br>

> 🤯 언리얼을 하기 위해 C++ 기억 되살리기 프로젝트

# 스마트 포인터

- unique_ptr
  - 진짜 많이 씀! 중요!
- shared_ptr
  - 밑의 weak와 묶여서 쓰임.
- weak_ptr



```cpp
// 예시: 포인터
#include "Vector.h"

int main()
{
    Vector* myVector = new Vector(10f, 30.f);

    // ...

    delete myVector;

    return 0;
}
```

이렇게 new 로 할당을 해줬으면 delete로 지워줘야 함.  
그런데 중간에 반환을 해버리는 경우 delete까지 코드가 가지 않아  
메모리가 누수가 발생하는 경우가 있을 수 있음.  

> 💡 스마트 포인터를 쓰면, delete를 직접 호출할 필요가 없다!  
> 그리고 가비지 컬렉션보다 빠르다!  
> ( 쓰이지 않는 순간에 딱 지워지기 때문에. )

## 1. unique_ptr

포인터 소유자가 하나 밖에 없다.  
나 말고 누구도 못쓰게 만드는 포인터.  
소유자가 한 명이면 그 사람이 지우면 된다.  

```cpp
#include <memory>
#include "Vector.h"

int main()
{
    std::unique_ptr<Vector> myVector(new Vector(10.f, 30.f));
    // 포인터 부호가 없음!

    myVector->Print();
    // 그러나 포인터처럼 동작할 수 있음.
    // 그리고 delete가 없음.

    return 0;
}
```

- 포인터(원시 포인터? 기존 포인터?) 를 단독으로 소유
- 소유한 원시 포인터는 누구하고도 공유되지 않았음
  - 따라서 <b>복사나 대입 불가능!</b>
- unique_ptr가 범위(scope)를 벗어날 때, 원시 포인터는 지워짐 (delete)
  ```cpp
  std::unique_ptr<Vector> myVector(new Vector(10.f, 30.f));

  std::unique_ptr<Vector> copiedVector1 = myVector;         // 컴파일 에러! 유니크 포인터 대입 안 됨.
  std::unique_ptr<Vector> copiedVector2(myVector);          // 컴파일 에러! 유니크 포인터 복사 못 함.
  ```

### 1-1. 유니크 포인트가 적합한 3가지 경우

#### 예시1: 클래스에서 생성자/소멸자

```cpp
// OLD
// Player.h
class Player
{
    private:
        Vector* mLocation;
};

// Player.cpp
Player::Player(std::string name)
        : mLocation(new Vector())
{

}

Player::~Player()
{
    delete mLocation;           // 소멸자에서 지워줬어야 했음
}
```

```cpp
// NEW!
// Player.h
class Player
{
    private:
        std::unique_ptr<Vector> mLocation;
};

// Player.cpp
Player::Player(std::string name)
        : mLocation(new Vector())
{

}

// 소멸자에 delete 키워드 안 씀!
// Player 소멸될 때 알아서 지워짐.
```

#### 예시2: 지역 변수

```cpp
// OLD
#include "Vector.h"
int main()
{
    Vector* vector = new Vector(10.f, 30.f);

    vector ->Print();

    delete vector;
}
```

```cpp
// NEW!
#include "Vector.h"
#include <memory>
int main()
{
    std::unique_ptr<Vector> vector(new Vector(10.f, 30.f));

    vector ->Print();
}
```

#### 예시3: STL 벡터에 포인터 저장하기

```cpp
// OLD
#include <vector>
#include "Player.h"

int main()
{
    std::vector<Player*> players;
    players.push_back(new Player("Lulu"));
    players.push_back(new Player("Coco"));

    for (int i = 0; i < players.size(); ++i)
    {
        delete players[i];
    }

    players.clear();
    // ...
}
```

```cpp
// NEW!
#include <vector>
#include "Player.h"

int main()
{
    std::vector<std::unique_ptr<Player>> playerList;

    playerList.push_back(std::unique_ptr<Player>(new Player("Lulu")));
    playerList.push_back(std::unique_ptr<Player>(new Player("Coco")));

    players.clear();
    // 클리어 하면서 소멸자 호출되며 다 지워짐.

    // 사실 굳이 clear호출 안 해도 됨.
    // main() 스코프 나가면서 vector 소멸자 불러오게 됨.
    // 자체적으로 clear 호출되면서 지워짐.
    // ...
}
```

### 1-2. 유니크 포인터 만들기 (C++14 이후)

#### 문제: 원시 포인터 공유

```cpp
Vector* vectorPtr = new Vector(10.f, 30.f);         // 원시 포인터 생성
std::unique_ptr<Vector> vector(vectorPtr);
// 대입해서 넣어줌

std::unique_ptr<Vector> anotherVector(vectorPtr);
// 대입해서 넣어줌 2. 원시 포인터라 유니크 포인터에 대입 가능 함 ㅎ;;
// 같은 포인터를 또 넣어주게 된 것. 소유자 권한 개념이 깨짐!
// 그래서 nullptr을 대입하여 없애버릴 예정.

anotherVector = nullptr;            // 이거 연산자 오버로딩임
// 그.러.나.
// 이건 유니크 포인터가 null이 되는게 아니라, 그 안에 있는 원시 포인터를 null로 해주는 것.
// 포인터에 대한 소유권을 놓는 것이므로, 자동으로 소멸자 호출

// 결국 anotherVector는 empty가 되버리고,
// vectorPtr 값이 날아가게 됨.
// 그런데 vector는 아직도 본인이 포인터를 제대로 갖고있는 줄 앎.

// 그래서 나중에 vector가 소멸될 때 소멸자를 호출하게 되고,
// vectorPtr는 이미 날라갔으므로 에러가 발생하게 될 것...
```

#### 해결책 (C++14 이후) : make_unique

```cpp
#include <memory>
#include "Vector.h"

int main()
{
    std::unique_ptr<Vector> myVector = std::make_unique<Vector>(10.f, 30.f);

    myVector->Print();

    return 0;
}
```

- std::make_unique()가 무슨 일을 할까?
  - 주어진 매개변수와 자료형으로 new 키워드 호출해줌
  - 둘 이상의 유니크 포인터가 원시 포인터를 공유할 수 없도록 막아줌

#### 만들기 예시

```cpp
Vector* vectorPtr = new Vector(10.f, 30.f);

// 싹 다 에러남!
std::unique_ptr<Vector> vector1 = std::make_unique(vectorPtr);
std::unique_ptr<Vector> vector2 = std::make_unique<Vector>(vectorPtr);
std::unique_ptr<Vector> vector3 = std::make_unique<Vector*>(10.f, 30.f);

// C++11
std::unique_ptr<Vector> vector(new Vector(10.f, 30.f));
std::unique_ptr<Vector[]> vectors(new Vector[20]);

// C++14 (이렇게 쓰자)
std::unique_ptr<Vector> vector = std::make_unique<Vector>(10.f, 30.f);
std::unique_ptr<Vector[]> vectors = std::make_unique<Vector[]>(20);
```

### 1-3. 유니크 포인터 재설정 : reset()

```cpp
int main()
{
    std::unique_ptr<Vector> vector = std::make_unique<Vector>(10.f, 30.f);
    vector.reset(new Vector(20.f, 40.f));
    vector.reset();                         // nullptr 대입하는 거랑 같다!
    // ...
}
```
- reset()을 호출하면 포인터를 교체한다
- 재설정 될 때, 소유하고 있던 원시 포인터는 자동으로 소멸 된다!
- nullptr을 꼭 써야 할까?
  - 아니다. 두 코드는 같음 (`vector = nullptr;`, `vector.reset();`)
  - nullptr이 reset()과 가독성이 더 높긴 함
  - 그러나 reset()은 vector가 원시 포인터가 아님을 확실히 보여줌

### 1-4. 유니크 포인터 가져오기 : get()

