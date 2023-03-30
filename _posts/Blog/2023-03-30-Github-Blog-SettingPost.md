---
title:  "Github Blog 환경 세팅, 로컬에서 블로그 미리보기 까지"
excerpt: "나중에 내가 다시 보려고 쓰는 포스트"

categories:
  - Blog
tags:
  - [Github, Blog]

toc: true
toc_sticky: false
 
date: 2023-03-29
last_modified_at: 2023-03-30
---

>이번에 하드 포맷하고 새로 윈도우 설치했다.  
>깃허브 블로그를 사용하기 위해 다시 환경을 세팅해줘야 했는데,
>나중에 또 해야 할 수도 있으므로 내가 참고하기 위해 작성하는 글!

## 1. Git 설치
[https://git-scm.com/download/win](https://git-scm.com/download/win)

기본이 되는 Git 부터 설치한다.
고민 없이 전부 Next를 쭉 눌러서 설치 했다.

## 2. Visual Studio Code 설치
[https://code.visualstudio.com/download](https://code.visualstudio.com/download)

블로그 글 작성을 위한(...) 코드를 설치한다.

## 3. Ruby 설치
[https://rubyinstaller.org/downloads/](https://rubyinstaller.org/downloads/)

위 링크로 들어가 루비를 다운받아 설치한다.  
WITH DEVKIT 의 굵은 글씨로 강조되어 있는 버전을 설치했다.  
캡쳐를 못했는데, <b>MSYS2</b> 옵션도 같이 설치하는 게 좋다! 그래야 이후에 설치할 <u>bundle</u> 을 원활히 사용할 수 있다.


MSYS2도 같이 선택했다면, 루비 설치 후 Finish 창이 열릴텐데
```
Run 'ridk install' to setup MSYS2 and development toolchain.
MSYS2 is required to install gems with C extensions.
```
이 문구에 체크가 되어 있는지 확인하고 Finish 를 누른다.  
그러면 MSYS2 설치를 위한 CMD 창이 열릴텐데, <b>1,2,3</b> 을 입력하고 엔터를 치면 MSYS2가 설치된다.

모든 설치가 끝나면 다시 엔터를 친다. 그럼 CMD 창이 종료된다.  
다시 CMD를 켜서 <b>ruby -v</b> 를 입력해보면  
설치된 루비의 버전이 뜰 것이다.

## 4. Jekyll, bundler 설치
여기서 나는 CMD 창을 <b>관리자 권한</b>으로 다시 켜주었다.  
그다음 CMD에서 아래 명령어를 입력하여 Jekyll를 설치했다.
```
gem install jekyll
```
그리고 bundler도 설치했다.
```
gem install bundler
```

여기까지 내가 설치한 플로우!

이후 블로그에 업로드 할 글을 작성 후 로컬에서 확인해보기 위해 로컬에서 블로그를 구동해보았다.  
내가 로컬에서 구동한 방법은 내가 Clone 받은 Git 경로로 들어가서  
해당 위치에서 Git Bash를 실행하여 아래 명령어를 쳐주었다.
```
bundle exec jekyll serve
```
명령어를 쳐 준 이후 인터넷창에서 <b> <u>localhost:4000</u> </b> (혹은 http://127.0.0.1:4000//)  
를 쳐주면 접속이 되어야 하는데...  
접속이 되지 않았다.

Bundler::GemNotFound 에러였다.  
그래서 열려있는 Git Bash 창에 아래 명령어를 쳐주었다.
```
bundler
```
치니까 뭔가 주르륵 설치되었다.  
완료 후엔 다시 로컬 구동을 위해 아래 명령어를 쳐주었다.
```
jekyll serve
```
그러니까 제대로 로컬 구동이 되었다!  
localhost:4000 로 접속도 되고!  
Git Bash 를 켜놓은 채 md파일을 작성하고 저장하면 Regenerating 되는걸 볼 수 있다.  
로컬 페이지를 새로고침 해보면, 로컬에서 작성한 글을 볼 수 있다.