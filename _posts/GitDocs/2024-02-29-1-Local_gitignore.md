---
title:  "로컬에서만 gitignore 적용하는 법"
excerpt: ".gitignore"

categories:
  - GitDocs
tags:
  - [Git]

toc: false
toc_sticky: false
 
date: 2024-02-29
last_modified_at: 2024-02-29
comments: true
---

<br>

보통 `.gitignore` 파일을 추가할 때 커밋&푸시를 하게 되니  
로그도 남고, 원격(Remote)으로도 파일 적용이 되는데...  

원격에 적용하지 않고 로컬에서만 ignore 할 수 있다.  

`클론 받은 레포지토리 폴더 / .git / info` 에 있는 `exclude` 파일을 수정하면 된다.  
이 파일을 메모장으로 열어보면,  

```
# git ls-files --others --exclude-from=.git/info/exclude
# Lines that start with '#' are comments.
# For a project mostly in C, the following would be a good set of
# exclude patterns (uncomment them if you want to use them):
# *.[oa]
# *~
```

이런 설명이 적혀있는데  
설명 아래에 .gitignore 내용을 작성해주면 된다.  