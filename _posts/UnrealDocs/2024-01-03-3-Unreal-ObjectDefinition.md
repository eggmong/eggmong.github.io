---
title:  "언리얼 메모 4 / 언리얼 오브젝트"
expert: "FName, FText"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: true
toc_sticky: true
 
date: 2024-01-03
last_modified_at: 2024-01-03
---

<br>


# 언리얼 오브젝트: UObject

- 언리얼 엔진이 설계한 새로운 시스템의 단위 오브젝트(객체)
  - 기존 C++ 오브젝트에 모던 객체 지향 설계를 위한 다양한 기능을 추가한 오브젝트
  - 일반 C++ 오브젝트와 언리얼 오브젝트의 두 객체를 모두 사용할 수 있음
  - 구분을 위해 일반 C++ 오브젝트는 `F`, 언리얼 오브젝트는 접두사 `U` 붙임

- 각 오브젝트의 사용 용도
  - C++ 오브젝트 : 저수준의 빠른 처리를 위한 기능 구현에 사용
  - 언리얼 오브젝트 : 콘텐츠 제작에 관련된 복잡한 설계 구현에 사용


[언리얼 오브젝트 문서](https://docs.unrealengine.com/5.3/ko/objects-in-unreal-engine/)



Object 클래스를 선택해서 MyObject라는 클래스를 새로 만들어보면  
헤더가 생기는데,  

```cpp
#pragma once

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "MyObject.generated.h"

/**
 * 
 */
UCLASS()
class UNREALOBJECT_API UMyObject : public UObject
{
	GENERATED_BODY()
	
};
```

> <b>UNREALOBJECT_API</b> : UnrealObject 라는 이름을 가진 프로젝트의 Api라고 지정하는 것  
> 이걸 지우면, MyObject는 다른 모듈에서 참조를 하지 않고  
> UnrealObject라는 모듈 내에서만 사용할 수 있게 됨  