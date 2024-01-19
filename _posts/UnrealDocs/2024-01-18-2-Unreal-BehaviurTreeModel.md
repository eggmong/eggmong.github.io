---
title:  "행동 트리 모델"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: true
toc_sticky: true
 
date: 2024-01-18
last_modified_at: 2024-01-18
---

<br>

# 행동 트리 모델

- 행동을 중심으로 설계
- 왼쪽에 우선순위를 부여해서 왼쪽에 배치한 노드를 먼저 처리하도록 설정
- 단 부모 노드에서 다수의 행동을 컨트롤 함. 이것을 컴포짓(Composit) 노드라고 함.
  - Selector 컴포짓 : 여러 행동 중 하나의 행동을 지정하여 수행하는 노드
  - Sequence 컴포짓 : 여러 행동을 연달아 수행하는 노드
  - Parallel 컴포짓 : 여러 행동을 함께 수행하는 노드

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/UnrealBehaviurTreeModel1.png)  



## 행동에 대한 다양한 결과

- 성공 (Succeded) : 행동이 성공했다고 알려주는 값
- 실패 (Failed) : 행동이 실패했다고 알려주는 값
- 중지 (Aborted) : 진행하고 있었는데 외부 요인으로 인한 행동의 실패를 알려주는 값
- 진행중 (InProgress) : 진행중이라고 컴포짓 노드에게 보고하는 값

![bt](https://github.com/eggmong/eggmongImages/raw/main/UnrealDocs/UnrealBehaviurTreeModel2.png)  


결과 값을 보고 받은 컴포짓 노드는 값에 따라 다른 행동을 취한다.  



## 행동 트리 모델의 구성 요소

컴포짓 노드에 부착하는 다양한 추가 기능  

- 데코레이터(Decorator) : 컴포짓 노드가 실행되는 조건을 지정
- 서비스(Service) : 컴포짓 노드가 활성화될 때 주기적으로 실행하는 부가 명령
- 관찰자 중단(Abort) : 데코레이터 조건에 부합되면 컴포짓 노드 내 활동 모두 중단

