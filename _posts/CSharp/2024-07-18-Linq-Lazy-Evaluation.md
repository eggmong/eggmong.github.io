---
title:  "[Linq] Lazy Evaluation (지연 실행)"


categories:
  - CSharp
tags:
  - [CSharp]

toc: false
toc_sticky: true
 
date: 2024-07-18
last_modified_at: 2024-07-18
---

<br>

지연 실행(계산) 이라는 개념을 알게 되어서 찾아봤다.  
그리고 기억해두기 위해 써놓는 글.  

```csharp
var list = new int[] { 10, 20, 30, 40, 45, 50 };
list.Where(x => x > 30)
    .Select(x => x * x);
```

list를 순회하며 30보다 큰 값만 골라낸 컬렉션을 만들고  
이 컬렉션을 순회하며 각각 제곱하는 코드이다- 라고 생각하겠지만,  
실제로 이 코드는 아무 동작도 하지 않는다.  

```csharp
var list = new int[] { 10, 20, 30, 40, 45, 50 };
list
  .Where(x => {
    Console.WriteLine($"[Where] {x}");
    return x > 30;
  })
  .Select(x => {
    Console.WriteLine($"[Select] {x}");
    return x * x;
  });
```

확인해보기 위해 위의 코드에서 로그를 찍은 코드이다.  
Where에서 x 값을 30과 비교하기 전에 x값을 출력하고, Select로 순회할 때 마다 x값을 출력하도록 한다.  





---

* 참고 링크  
[https://medium.com/@qjfrntop12/linq-파헤치기-bc2c60dfca4f](https://medium.com/@qjfrntop12/linq-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0-bc2c60dfca4f)