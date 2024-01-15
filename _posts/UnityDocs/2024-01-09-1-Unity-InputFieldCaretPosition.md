---
title:  "[UI] InputField caret(커서) 위치 재설정 : MoveTextEnd"
expert: ""

categories:
  - UnityDocs
tags:
  - [Unity, Game Engine, UI, InputField]

toc: true
toc_sticky: true
 
date: 2024-01-05
last_modified_at: 2024-01-05
---

<br>

회원가입 시스템 작업 중,  
가입 단계별로 팝업이 뜨고 거기에 유저 정보를 입력하는 UI 작업을 하고 있었다.  

이전 팝업으로 되돌아갈 경우  
이전 팝업에 입력했던 InputField 내용들은 지워지도록 구현했었다.  

그런데 내용은 지워졌어도 해당 InputField를 눌러보면  
이전에 입력했던 내용의 끝 부분에 caret(커서) 가 깜빡이는 것이었다.  
다시 누르면 올바르게 첫 부분으로 돌아가긴 하지만.  

이걸 고치려면  
InputField 초기화 해줄 때 `MoveTextEnd` 를 호출해주면 된다.  

```cpp
for (int i = 0; i < inputFields.Length; i++)
{
    inputFields[i].text = "";
    inputFields[i].MoveTextEnd(false);

    // ...
}
```

- 유니티 문서
[https://docs.unity3d.com/kr/530/ScriptReference/UI.InputField.MoveTextEnd.html](https://docs.unity3d.com/kr/530/ScriptReference/UI.InputField.MoveTextEnd.html)