---
title:  "UPROPERTY 안의 인자 정리중..."

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: true
toc_sticky: true
 
date: 2024-01-14
last_modified_at: 2024-01-14
---

<br>

- Transient
  - 하단 예시는 StatComponent 작성 중의 예시인데, 이런 컴포넌트는 저장할 때 디스크에 저장이 됨
  - 그런데 CurrentHp 같은 건 게임을 할 때 마다 새롭게 지정되기 때문에 디스크에 저장할 필요가 없음
    - CurrentHp 같은 스탯 및 CurrentLevel 등...
  - 이 키워드를 사용해 <b>디스크에 저장을 하지 않도록</b> 하여 메모리 낭비 방지

- VisibleInstanceOnly
  - 인스턴스 마다 별도로 수행...
  
  ```cpp
  // 배치된 캐릭터들 마다 각각 다른 Hp 값을 설정할 수 있게 됨
  // 마치 Unity의 SerializedField 처럼?
  
  protected:
	UPROPERTY(VisibleInstanceOnly, Category = "Stat")
	float MaxHp;

	UPROPERTY(Transient, VisibleInstanceOnly, Category = "Stat")
	float CurrentHp;
  ```

