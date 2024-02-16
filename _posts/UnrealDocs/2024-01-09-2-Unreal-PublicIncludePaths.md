---
title:  "[정리] PublicIncludePaths.AddRange 코드의 의미"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: true
toc_sticky: true
 
date: 2024-01-09
last_modified_at: 2024-01-09
---

<br>



`PublicIncludePaths.AddRange` 는 언리얼 엔진의 빌드 시스템에서 사용되는 코드 일부입니다.  
이 코드는 C++ 프로젝트에서 헤더 파일의 검색 경로를 설정하는 데 사용됩니다.  

`PublicIncludePaths` 는 프로젝트의 public 헤더 파일을 참조하기 위한 경로 목록을 나타냅니다.  
`AddRange` 함수는 목록에 여러 경로를 추가하는 데 사용됩니다.  

따라서 `PublicIncludePaths.AddRange` 를 사용하여 프로젝트에 추가적인 공개 헤더 파일 경로를 정의할 수 있습니다.  
이렇게 함으로써 컴파일러가 헤더 파일을 찾을 때 지정된 경로도 검색 대상에 포함됩니다.  


```cpp
PublicIncludePaths.AddRange(
    new string[] {
        "MyProject/PublicFolder",
        "ThirdPartyLibrary/Include",
        // 추가적인 경로들...
    }
);
```

이는 "MyProject/PublicFolder" 및 "ThirdPartyLibrary/Include"와 같은 경로를  
PublicIncludePaths에 추가하여 해당 경로에 있는 헤더 파일을 포함할 수 있게 합니다.