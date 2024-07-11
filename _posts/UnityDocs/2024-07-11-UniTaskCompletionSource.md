---
title:  "[UniTask] UniTaskCompletionSource 사용하여 Task 기다리기"
expert: ""

categories:
  - UnityDocs
tags:
  - [Unity, Game Engine, UniTask]

toc: true
toc_sticky: true
 
date: 2024-07-11
last_modified_at: 2024-07-11
---

<br>

```csharp
UniTaskCompletionSource completionSource = new UniTaskCompletionSource();
_newNextCloseButton.OnClickEvent().Subscribe( _ =>
{
    completionSource.TrySetResult();
});

await completionSource.Task;
```

버튼의 클릭 이벤트가 발생할 때까지 기다렸다가,  
이벤트가 발생하면 비동기 작업이 완료되는 코드이다.  
하나하나 뜯어보자면,  

### UniTaskCompletionSource 생성

```
UniTaskCompletionSource completionSource = new UniTaskCompletionSource();
```

UniTaskCompletionSource 객체를 생성한다.  
이 객체는 나중에 특정 작업이 완료되었음을 알리기 위해 사용된다.  

### 버튼 클릭 이벤트 구독

```
_newNextCloseButton.OnClickEvent().Subscribe(_ =>
{
    completionSource.TrySetResult();
});
```

_newNextCloseButton의 클릭 이벤트를 구독(subscribe) 한다.  
버튼이 클릭되면, 이벤트 핸들러가 호출된다.  
이 이벤트 핸들러에서는 completionSource.TrySetResult()를 호출하여  
UniTaskCompletionSource의 작업을 완료 상태로 설정한다.  

### 비동기 작업 대기

```
await completionSource.Task;
```

completionSource.Task는 비동기 작업을 나타내며,  
await 키워드를 사용하여 이 작업이 완료될 때까지 비동기적으로 대기한다.  

그 뒤 클릭 이벤트 핸들러가 호출되면, completionSource.TrySetResult()에 의해 이 작업이 완료 상태로 설정되고, await가 종료된다.  