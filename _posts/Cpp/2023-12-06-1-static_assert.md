---
title:  "새로운 키워드 (C++11~) / static_assert, default/delete, final/override"

categories:
  - Cpp
tags:
  - [Cpp, C++, static_assert, default, delete, final, override]

toc: true
toc_sticky: true
 
date: 2023-12-06
last_modified_at: 2023-12-06
---

<br>

> 🤯 언리얼을 하기 위해 C++ 기억 되살리기 프로젝트

# 새로운 키워드 (C++11~) / static_assert, default, delete

기존의 assert는 실행 도중에, 조건을 만족하지 않았을 때 실행을 멈춰주고 프로그래머만 볼 수 있게 했었음.  
디버그 빌드에서만 도는 것.  
그러나 `static_assert`는 컴파일 때, 컴파일 도중에 assert를 할 수 있게 함.  
실행 도중이 아니라 컴파일 중에 문제를 해결할 수 있게 해줌.  



## 1. static_assert 키워드

- assert
  - 실행 중에 가정(assertion)이 부합하는지 체크
  - 오직 디버그 빌드에서만 작동
    - release 빌드에선 작동하지 않음 -> 개발 도중에만 적중한다는 것...
  - 실패한 assert를 보려면 반드시 프로그램을 실행해야 함
- static_assert
  - 컴파일 중에 어서션 평가
    - 컴파일러가 알 수 없는 조건일 경우에도 에러 출력
  - 컴파일러가 assert 조건이 참인지 아닌지 앎
  - 실패하면 컴파일러는 컴파일 에러 출력



## 2. default/delete

### 2-1. default 키워드

```cpp
// Dog.h
class Dog
{
    public:
        Dog() = default;                    // 컴파일러가 Dog를 대신 만들어냄!
        Dog(std::string name);
    private:
        std::string mName;
};

// Dog.cpp
#include "Dog.h"

// 원래 여기에
//Dog::Dog()
//{

//}
// 이렇게 기본생성자가 생성되어야 하는데, default를 적어둬서
// 컴파일러가 Dog를 대신 만들어냄!

Dog::Dog(std::string name)
{
    // ...
}
```

- `default` 키워드를 사용하면, 컴파일러가 특정한 생성자, 연산자 및 소멸자를 만들어 낼 수 있음
- 그래서, 비어 있는 생성자나 소멸자를 구체화할 필요가 있음
- 또한 기본 생성자, 연산자 및 소멸자를 더 분명하게 표현할 수 있음
- 컴파일러가 봤을 때 `default` 키워드 붙이나마나 어차피 기본 생성자는 만들어 내야하니 똑같은 상황임


### 2-2. delete 키워드

```cpp
// Dog.h
class Dog
{
    public:
        Dog() = default;                            // 기본생성자는 default
        Dog(const Dog& other) = delete;             // 복사생성자 삭제
};

// Main.cpp
#include "Dog.h"
int main()
{
    Dog myDog;
    Dog copiedMyDog(myDog);                         // 에러 발생할 것. delete로 지웠으니까
}
```

- 컴파일러가 자동으로 생성자를 만들어 주길 원치 않을 때 사용하는 키워드


## 3. final/override

### 3-1. final 키워드

- 상속을 막을 수 있다.

```cpp
// Animal.h
class Animal final                  // 상속 막겠다는 키워드
{
    public:
        virtual void SetWeight(int weight);
        // ...
};

// Dog.h
#include "Animal.h"
class Dog : public Animal           // 에러 발생
{
    // ...
}
```

- 또는 재상속을 막을 수도 있다.  

```cpp
// Dog.h ---------------------------
#include "Animal.h"

// Animal 클래스는 final 이 아니라 상속 가능한 상태!
class Dog final : public Animal
{
    public:
        virtual void SetWeight(int weight) override;
        // ...
};

// Corgi.h ---------------------------
#include "Dog.h"

class Corgi : public Dog            // 에러 발생
{
    // ...
};
```

- 함수의 상속도 막을 수 있다. (함수의 재정의를 막는다는 의미이기도 함)

```cpp
// Animal.h ---------------------------
class Animal
{
    public:
        virtual void SetWeight(float weight) final;
        // ...
};


// Dog.h ---------------------------
#include "Animal.h"

class Dog : public Animal
{
    public:
        virtual void SetWeight(float weight) override;        // 에러 발생!
        Dog();
        ~Dog();
        // ...
}
```

> 💡 final 키워드 정리
> - 클래스나 가상 함수를 파생 클래스에서 오버라이딩 못하게 함
> - 컴파일 도중에 확인함.
> - 당연히 가상 함수가 아니면 쓸 수 없음
>   - 가상 함수 아니면 재정의 의미가 없으니까 ;;



### 3-2. override 키워드

#### 잘못된 가상 함수 오버라이딩

```cpp
// Animal.h ---------------------------
class Animal
{
    public:
        virtual void SetWeight(float weight);
        void PrintAll();
        // ...
};


// Dog.h ---------------------------
#include "Animal.h"

class Dog : public Animal
{
    public:
        virtual void SetWeight(int weight);
        // Animal::SetWeight(float weight)를 오버라이딩 하지 않음!
        // 잘못된 함수 작성임.
        // 그런데 컴파일러가 문제라고 인식 못 함.
        // '새 함수 작성한 것일 수도 있잖아 ㅇㅅㅇ'

        // 저 문제를 해결하기 위한 방법 : override 키워드
        virtual void SetWeight(float weight) override;
        // 부모한테 있는 함수를 재정의 할거야 <- 라는 명확한 키워드

        virtual void SetWeight(int weight) override;
        // 컴파일 에러 출력해줌

        void PrintAll() override;
        // 컴파일 에러 출력해줌
        // 가상 함수가 아니니까
};
```

> 💡 override 키워드 정리
> - 잘못된 가상 함수 오버라이딩을 막으려면 `override` 키워드 사용
>   - 잘못된 오버라이딩일 경우 컴파일 에러 출력
> - 당연히 가상 함수가 아니면 쓸 수 없음
>   - 컴파일 에러 출력
> - 이것도 컴파일 도중에 검사함!

