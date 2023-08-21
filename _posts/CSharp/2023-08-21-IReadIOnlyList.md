---
title:  "IReadOnlyList 인터페이스"

categories:
  - CSharp
tags:
  - [CSharp]

toc: false
toc_sticky: true
 
date: 2023-08-21
last_modified_at: 2023-08-21
---

<br>

List의 항목들을 읽어오기만 할 수 있는 컬렉션이라고 한다.  
그래서 Add, Clear 등의 함수가 존재하지 않는다.  


나의 경우 기존엔 외부 클래스에서 List를 Clear 해주고 있었는데  
이런 형태는 위험하다고 한다.  
아무래도 외부에서 제어가 가능한 게 좋지 않은 거니까...  

```c#
private List<Sprite> stageSprites;
public IReadOnlyList<Sprite> StageSprites => this.stageSprites;
```

그래서 내 코드를 이렇게 수정하게 되었다.  
접근만 `IReadOnlyList` 타입으로 선언하여 외부에서 Add나 Clear를 못하도록 해주었다.  




***
참고 링크
- [https://learn.microsoft.com/ko-kr/dotnet/api/system.collections.generic.ireadonlylist-1?view=net-7.0](https://learn.microsoft.com/ko-kr/dotnet/api/system.collections.generic.ireadonlylist-1?view=net-7.0)