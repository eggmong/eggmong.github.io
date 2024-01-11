---
title:  "[Refactoring] 조건이 길어질 땐 변수로 빼자, 상수 값도 변수로 빼자"


categories:
  - CSharp
tags:
  - [CSharp]

toc: false
toc_sticky: true
 
date: 2023-09-12
last_modified_at: 2023-09-12
---

<br>

긴 말 할 것 없이 코드로 가자...  
내 이전 코드는 이랬다.  

```c#
...

targetArray = this.backTiles
    .Where(tile => 
        (tile.position.x >= 1 && tile.position.x <= 6 && (tile.position.y == 0 || tile.position.y == 8)) ||
        ((tile.position.x == 1 || tile.position.x == 6) && (tile.position.y >= 0 && tile.position.y <= 8))
    )
    .ToArray();

...
```

저 상태로 코드 리뷰를 보냈다가 [@BeautyfullCastle](https://github.com/BeautyfullCastle) 님께 댓글로 피드백을 받았다.  

![log3](https://github.com/eggmong/eggmongImages/raw/main/CSharp/2023-09-12-CodeRefactoring.png)  

상수 값은 변수로 빼서 이름에 설명을 담는게 이해 하기 쉽다. (이 값이 어디서 튀어나온건지 맥락도 파악할 수 있고)  

또한 조건이 길어질 경우 변수로 빼서 변수로 비교하는 게 가독성이 좋다.  

```c#
...

// X와 Y가 특정 범위에 있는 타일
bool isXInRange = (tile.position.x >= (float)Define.MapSize.In_XStart && tile.position.x <= (float)Define.MapSize.In_XEnd);
bool isYInRange = (tile.position.y >= (float)Define.MapSize.In_YStart && tile.position.y <= (float)Define.MapSize.In_YEnd);

// X 또는 Y가 경계(특정 숫자)에 있는 타일
bool isOnBoundaryX = (Mathf.Approximately(tile.position.x, (float)Define.MapSize.In_XStart) ||  // 1f
                      Mathf.Approximately(tile.position.x, (float)Define.MapSize.In_XEnd));     // 6f
                    
bool isOnBoundaryY = (Mathf.Approximately(tile.position.y, (float)Define.MapSize.In_YStart) ||  // 0f
                      Mathf.Approximately(tile.position.y, (float)Define.MapSize.In_YEnd));     // 8f

bool meetsCondition = (isOnBoundaryX && isYInRange) || (isXInRange && isOnBoundaryY);

return meetsCondition;
```

피드백을 받고 바꿔본 코드.  

