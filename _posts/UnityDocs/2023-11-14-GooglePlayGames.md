---
title:  "[GooglePlayGames] GooglePlayGames 연동하면서 겪은 이슈 1"
expert: "IOS 빌드 모듈 설치 안해서 생긴 이슈, path 폴더 이름 변경 이슈"

categories:
  - UnityDocs
tags:
  - [Unity, Game Engine, GooglePlayGames]

toc: true
toc_sticky: true
 
date: 2023-11-14
last_modified_at: 2023-11-14
---

<br>

일단 유니티에 GooglePlayGames 연동하려면  
[Github Link(클릭)](https://github.com/playgameservices/play-games-plugin-for-unity)에서 sdk 다운받아서 패키지 실행해주면 된다.  
(패키지 경로 : play-games-plugin-for-unity-master/current-build 폴더 안에 있음)  

그리고 임포트 해주면 된다.

![import1](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-11-14-GooglePlayGames/GooglePlayGamesPluginDependencies%EA%B2%BD%EB%A1%9C%EB%B3%80%EA%B2%BD1.png){: width="60%" height="60%"}  

![import2](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-11-14-GooglePlayGames/GooglePlayGamesPluginDependencies%EA%B2%BD%EB%A1%9C%EB%B3%80%EA%B2%BD2.png)  

이 창이 안 뜬다면 Build Setting을 Android로 바꾸면 된다.  

그리고 겪은 이슈를 해결했다.  


# 1. IOS 빌드 모듈 설치 안해서

임포트 하고 나서 이런 Error log가 나타났다.  

![iOS1](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-11-14-GooglePlayGames/ios%EB%AA%A8%EB%93%88%EC%84%A4%EC%B9%98%EC%95%88%EB%90%98%EC%96%B4%EC%84%9C1.png)  

> 'Assets/ExternalDependencyManager/Editor/1.2.169/Google.IOSResolver.dll' will not be loaded due to errors:
> Unable to resolve reference 'UnityEditor.iOS.Extensions.Xcode'.

IOS 빌드 모듈 설치 해주면 해결된다.  

![iOS2](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-11-14-GooglePlayGames/ios%EB%AA%A8%EB%93%88%EC%84%A4%EC%B9%98%EC%95%88%EB%90%98%EC%96%B4%EC%84%9C2.png)  



# 2. 파일의 path 변경

임포트 다 된 줄 알았는데 이번엔 Warning log가 나타났다.

![path](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-11-14-GooglePlayGames/GooglePlayGamesPluginDependencies%EA%B2%BD%EB%A1%9C%EB%B3%80%EA%B2%BD3.png)  
![path](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-11-14-GooglePlayGames/GooglePlayGamesPluginDependencies%EA%B2%BD%EB%A1%9C%EB%B3%80%EA%B2%BD4.png)  

> Assets/GooglePlayGames/com/google.play.games/Editor/GooglePlayGamesPluginDependencies.xml:11:
> Repo path 'Packages/com.google.play.games/Editor/m2repository' does not exist.

로그에 적혀있는대로 `GooglePlayGamesPluginDependencies.xml` 파일을 열어봤다.  

![path](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-11-14-GooglePlayGames/GooglePlayGamesPluginDependencies%EA%B2%BD%EB%A1%9C%EB%B3%80%EA%B2%BD5.png)  

경로가 `Packages/com.google.play.games/...` 로 되어 있었다.  

![path](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-11-14-GooglePlayGames/GooglePlayGamesPluginDependencies%EA%B2%BD%EB%A1%9C%EB%B3%80%EA%B2%BD6.png)  

그런데 임포트한 패키지에선 `GooglePlayGames/com.google.play.games/...` 로 되어 있었다.  

![path](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-11-14-GooglePlayGames/GooglePlayGamesPluginDependencies%EA%B2%BD%EB%A1%9C%EB%B3%80%EA%B2%BD7.png)  

그래서 파일의 경로 바꿔주고 저장해보니 Warning log가 뜨지 않았다.  
그런데 이게 올바른 해결 방법인지는 잘 모르겠다.  