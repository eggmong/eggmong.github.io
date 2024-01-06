---
title:  "언리얼 메모 7 / 설계 - 컴포지션"
expert: "컴포지션, Composition"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: true
toc_sticky: true
 
date: 2024-01-05
last_modified_at: 2024-01-05
---

<br>

> 컴포지션은 객체 지향 설계에서 Has-A 관계를 구현하는 설계 방법

# 컴포지션 구현 방법

- 한 언리얼 오브젝트에는 항상 클래스 기본 오브젝트 CDO가 있다.
- 언리얼 오브젝트간의 컴포지션은 어떻게 구현할 것인가?
  - 1. CDO에 미리 언리얼 오브젝트를 생성해 조합한다. (필수적 조합)
  - 2. CDO에 빈 포인터만 넣고 런타임에서 언리얼 오브젝트를 생성해 조합한다. (선택적 조합)
- 언리얼 오브젝트를 생성할 때 컴포지션 정보를 구축할 수 있다.
  - 내가 소유한 언리얼 오브젝트를 Subobject라고 한다.
  - 나를 소유한 언리얼 오브젝트를 Outer라고 한다.

![Composition](https://drive.google.com/uc?export=view&id=1WjzaxoI1wgotb8Kqo5K3Tcjqc7mOq02E)  


## 코드


```cpp

```



### 언리얼5에서 오브젝트 포인터 가이드

[언리얼5 마이그레이션 문서](https://docs.unrealengine.com/5.0/ko/unreal-engine-5-migration-guide/#c++%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8%ED%8F%AC%EC%9D%B8%ED%84%B0%ED%94%84%EB%A1%9C%ED%8D%BC%ED%8B%B0)

TObjectPtr 로 포인터를 감싸라고 한다.

