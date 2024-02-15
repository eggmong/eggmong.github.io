---
title:  "[Unity] AssetPostprocessor - 에셋 임포트 설정 오버라이드를 쉽게!"
categories:
  - UnityDocs
tags:
  - [Unity, Game Engine, Asset, Asset Import]

toc: true
toc_sticky: true
date: 2023-05-20
last_modified_at: 2023-05-20
---

<br>

*유니티 GDC 2023 영상을 보다가 흥미로운 항목들은 정리해보는 중!*

> 💡 개발 중에 에셋을 임포트 할 때, 임포트 설정을 해야 하는 경우가 있다.  
![Import Setting]({{ site.imageurl }}UnityDocs/2023-05-20_AssetPostprocessor_1.png){: width="60%" height="60%"}  
> 에셋들을 하나하나 일일이 설정하다 보면 (귀찮고) 실수할 수 있기 때문에,  
> `AssetPostprocessor` 를 사용하여 쉽게 임포트 설정을 적용할 수 있다.  

## 사용 방법

`AssetPostprocessor` 를 상속하면 콜백을 받을 수 있는데, 에셋의 타입과 콜백을 받으려는 시점에 따라 함수를 구현해주면 된다.  

```c#
using UnityEngine;
using UnityEditor;

public class AssetPostprocessorTest : AssetPostprocessor
{
  ...
}
```
    
나는 `Texture`를 사용해 테스트 코드를 작성해보았다.  

<br>

### OnPreprocessTexture

텍스쳐 임포터가 실행되기 직전에 콜백을 받을 수 있는 함수이다.  
텍스쳐의 압축 형식 같은 임포트 설정 값을 지정할 수 있다.  

```c#
private void OnPreprocessTexture()
    {
        // 임포트한 에셋 경로 출력
        Debug.Log($"OnPreprocessTexture ---> {assetPath}");

        // 에셋이 ImportTest 폴더에 있는 경우에만 로그 출력 및 처리
        string lowerCaseAssetPath = assetPath.ToLower();
        if (lowerCaseAssetPath.IndexOf("/importtest/") == -1)
        {
            return;
        }

        TextureImporter importer = (TextureImporter)assetImporter;
        importer.textureCompression = TextureImporterCompression.Uncompressed;

        Debug.Log($"textureCompression : {importer.textureCompression}");
    }
```

<br>

### OnPostprocessTexture

텍스쳐 가져오기가 완료되면 콜백을 받을 수 있는 함수이다.  

```c#
private void OnPostprocessTexture(Texture2D texture)
    {
        // 임포트한 에셋 경로 출력
        Debug.Log($"OnPostprocessTexture ---> {assetPath}");

        // 에셋이 ImportTest 폴더에 있는 경우에만 로그 출력 및 처리
        string lowerCaseAssetPath = assetPath.ToLower();
        if (lowerCaseAssetPath.IndexOf("/importtest/") == -1)
        {
            return;
        }

        TextureImporter importer = (TextureImporter)assetImporter;

        Debug.Log($"textureType : {importer.textureType}");
    }
```

<br>

### OnPostprocessAllAssets

모든 에셋들의 변경 내역을 콜백으로 받아볼 수 있다.  
위 함수들과의 차이점은 `static` 으로 선언해야 한다는 것이다.  

```c#
    private static void OnPostprocessAllAssets(string[] importedAssets, string[] deletedAssets, string[] movedAssets, string[] movedFromAssetPaths)
    {
        foreach (string str in importedAssets)
        {
            // 임포트 한 에셋의 내역들이 로그로 출력된다.
            Debug.Log("Imported Asset: " + str);
        }
        foreach (string str in deletedAssets)
        {
            // 에셋을 지우는 경우에도 로그를 출력할 수 있다.
            Debug.Log("Deleted Asset: " + str);
        }

        for (int i = 0; i < movedAssets.Length; i++)
        {
            // 에셋의 경로가 변경된 경우도 잡을 수 있다.
            Debug.Log("Moved Asset: " + movedAssets[i] + " from: " + movedFromAssetPaths[i]);
        }
    }
```

<br>

#### 참고 링크 & 영상
- [[유니티 TIPS] 유니티가 추천하는 GDC 2023 주요·인기 세션 알아보기!](https://youtu.be/nfox34gUFTE)
- [https://docs.unity3d.com/ScriptReference/AssetPostprocessor.html](https://docs.unity3d.com/ScriptReference/AssetPostprocessor.html)
- [https://everyday-devup.tistory.com/80](https://everyday-devup.tistory.com/80)
- [https://docs.unity3d.com/kr/current/Manual/BestPracticeUnderstandingPerformanceInUnity4.html](https://docs.unity3d.com/kr/current/Manual/BestPracticeUnderstandingPerformanceInUnity4.html)