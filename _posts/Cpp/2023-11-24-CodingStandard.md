---
title:  "코딩 표준 etc"

categories:
  - Cpp
tags:
  - [Cpp, C++]

toc: true
toc_sticky: true
 
date: 2023-11-24
last_modified_at: 2023-11-24
---

<br>

> 🤯 언리얼을 하기 위해 C++ 기억 되살리기 프로젝트

# 코딩 표준 etc

## 읽기전용 매개변수를 상수화 하자

```cpp
bool TryDivide(Vector& result, const Vector& a, const Vector& b)
{
	const Vector a;
	const Vector b;
	Vector result;
	
	TryDivide(a, b, result);		// Compile Error
	TryDivide(result, a, b);
}
```

위의 코드에서 내부에서 값을 변경하지 않는다는 걸 알 수 있으니, 출력값이 어떤 변수인지 알 수 있음.  
하지만 넣어주는 변수값을 const로 지정 안해서 햇갈릴 수 있음...  

```cpp
bool TryDivide(Vector& result, const Vector& a, const Vector& b);
Vector a;
Vector b;
Vector c;
TryDivide(a, b, c);		// 어떤게 출력 결과인지 알 수 없음
```

그래서!  
> 읽기전용 매개변수는 상수 참조로  
> 출력결과용 매개변수는 포인터로  

`TryDivide(&a, b, c);`

a는 결과니까 주소 불러오는 연산자로...  

근데 간혹 NULL을 집어넣는 경우가 있을 수 있음  

`TryDivide(NULL, b, c);`

> 그래서 assert 함수를 사용해서 NULL 방지