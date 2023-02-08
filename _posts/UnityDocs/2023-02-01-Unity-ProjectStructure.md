---
title:  "[Unity] 유니티 프로젝트 구조, Assets 폴더 구조"

categories:
  - UnityDocs
tags:
  - [Unity, Game Engine]

toc: false
toc_sticky: false
 
date: 2023-02-01
last_modified_at: 2023-02-08
---

# 유니티 프로젝트 구조  
## 유니티 폴더 구조

먼저 유니티 프로젝트를 구성하는 기본적인 폴더는 <mark style='background-color: #fff5b1'>Assets, Packages, ProjectSettings</mark> 이다.  
SVN이나 Git 같은 VCS로 프로젝트를 관리할 때도 이 3가지 폴더만 업로드하여 관리하면 된다.


  
|폴더 이름|설명|
|:----|:---------------|
|Assets 폴더|프로젝트의 리소스(아트, 스크립트 등)들이 들어가는 폴더|
|Packages|프로젝트를 구성하는 패키지가 들어가는 폴더|
|ProjectSettings|프로젝트 환경셋팅 파일들이 들어가는 폴더.<br>씬들을 추가하는 Build Setting 도 여기에 들어가있다. (EditorBuildSettings.asset)|
|Library|Assets 폴더에 리소스를 추가하면, 유니티는 Assets 폴더에 있는 원본은 수정하지 않고 이 파일들을 읽어서 게임용 캐시 데이터 버전으로 변환한다. <br> 그리고 변환한 캐시 데이터들은 Library 폴더에 저장된다. <br> Library 폴더의 파일들은 Assets 폴더의 리소스들과 리소스의 meta 파일만 있으면 다시 생성할 수 있다. <u>그러므로 굳이 SVN에 업로드할 필요가 없다.</u>|
|obj|아직 연결되지 않은 컴파일된 바이너리 파일이 저장되는 폴더|
|Logs|유니티에서 기록된 Log들이 저장되는 폴더|
  
  
  
~~이전 회사에서 Library, obj 폴더는 물론이고 csproj, sln 파일까지 싹 다 svn에 커밋해놓은 걸 보고 뒷골이 땡긴 적이 있었다..~~  
  
   
*참고 링크  
- [https://drehzr.tistory.com/1306](https://drehzr.tistory.com/1306)  
- [https://forum.unity.com/threads/is-it-safe-to-delete-library-folder-of-backed-up-project.982032/](https://forum.unity.com/threads/is-it-safe-to-delete-library-folder-of-backed-up-project.982032/)  
- [https://stackoverflow.com/questions/5308491/what-are-the-obj-and-bin-folders-created-by-visual-studio-used-for](https://stackoverflow.com/questions/5308491/what-are-the-obj-and-bin-folders-created-by-visual-studio-used-for)  
- [https://answers.unity.com/questions/993291/is-library-folder-necessary.html](https://answers.unity.com/questions/993291/is-library-folder-necessary.html)  
- [https://docs.unity3d.com/560/Documentation/Manual/BehindtheScenes.html](https://docs.unity3d.com/560/Documentation/Manual/BehindtheScenes.html)  
- [https://docs.unity3d.com/Manual/ExternalVersionControlSystemSupport.html](https://docs.unity3d.com/Manual/ExternalVersionControlSystemSupport.html)  
  
  
## 에셋 폴더 구조 (Assets Folder Structure)

개인적인 생각이지만 주요한 씬들은 각각 폴더별로 나누어 관리를 하고,  
그 외에는 용도별로 Core, Common 등으로 구분하여 넣어두는게 보기 편한 것 같다.  
폴더가 너무 많아지면 난잡해보이니까...
  
![AssetsStructure](/assets/images/posts/assetsfolderstructure.png)
  
  
*참고 링크
- [https://sam-16930.medium.com/unity-project-structure-a694792cefed](https://sam-16930.medium.com/unity-project-structure-a694792cefed)  