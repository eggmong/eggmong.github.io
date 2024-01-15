---
title:  "[Error] [UniRx] bool 변수 여러개 구독하기 및 구조 재정립"
expert: "Observable, ReactiveProperty"

categories:
  - UnityDocs
tags:
  - [Unity, Game Engine, UI, UniRx]

toc: true
toc_sticky: true
 
date: 2024-01-05
last_modified_at: 2024-01-05
---

<br>

# 문제점

먼저 기반 설명과 잘못 생각한 구조를 서술하겠음.  

회원가입 시스템 작업을 하고 있는데,  
가입 단계별로 뜨는 팝업에서 요구하는 각 유저 정보를 입력하고,  
유저 정보를 입력하면 `다음 버튼` 이 활성화 되어 누르면 다음 단계 팝업이 뜨는 형태이다.  

여기서 나는 UniRx를 사용하여
1. 각 항목별 `InputField` 의 `onValueChanged` 이벤트를 구독하여  
   `InputField` 변수들의 값이 모두 `IsNullOrEmpty` 이 아니고,  
2. 메일 인증 확인이 되고(`isEmailCheck`), 비밀번호를 2번 정확히 썼다면(`isPasswordCheck`)  

다음 버튼이 활성화 되도록 작업했는데...  
했다고 생각했는데...

다음이 문제의 코드이다.  

```csharp
Observable.CombineLatest(inputFields.Select(field => field.onValueChanged.AsObservable()))
            .Subscribe(values =>
            {
                nextButton.interactable = values.All(value => !string.IsNullOrEmpty(value)) &&
                isEmailCheck && isPasswordCheck;
            })
            .AddTo(this);
```

<b>이 코드의 최대 문제점은</b>

`InputField` 들의 값들이 변하는 이벤트(`onValueChanged`)가 생겨야만 버튼이 활성화 된다는 것이다. ㅎㅎ...  
비밀번호를 먼저 올바르게 다 작성하고, 이메일 인증을 이후에 한다면  
인증 확인(`isEmailCheck`) 절차는 <b>버튼 클릭 이벤트</b>로 완료되도록 구현했기 때문에  
최종적으로 `다음 버튼`이 활성화 되지 않는다.


# 해결 방법

해결 방법은 간단하다.  
`isEmailCheck` 변수와 `isPasswordCheck` 변수를 `BoolReactiveProperty` 타입으로 바꾸고  
저 변수들을 구독하여 2개 다 true가 되는 시점에 `다음 버튼` 을 활성화 시켜 주면 된다.  

```csharp
isEmailCheck.CombineLatest(isPasswordCheck, (x, y) => x && y)
            .Subscribe(allTrue => nextButton.interactable = allTrue)
            .AddTo(this);
```