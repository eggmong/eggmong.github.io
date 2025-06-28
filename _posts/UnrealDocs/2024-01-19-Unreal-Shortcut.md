---
title:  "[정리] UE5 단축키"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: true
toc_sticky: true
 
date: 2024-01-19
last_modified_at: 2024-01-19
---

<br>


# UE5 단축키 정리

## 에디터

- 복사하기
  - `Ctrl + C`

- `Ctrl + N`
  - 새 레벨 생성

- `Ctrl + Alt + F11`
  - 라이브 코딩 빌드
    변경된 코드 내용을 에디터에 바로 적용해주는 것.  

* 핫리로드: vs 컴파일이 끝나면 그 내용을 에디터에 적용해주는 것
라이브 코딩은 라이브 코드를 이용해서만 컴파일 함.
(맞는지 확인 필요)



### 뷰포트

- `우클릭` 누른채 키를 입력하면 뷰포트 이동 가능
  - [공식 문서 링크](https://docs.unrealengine.com/4.27/ko/BuildingWorlds/LevelEditor/Viewports/ViewportControls/#%EA%B2%8C%EC%9E%84%EC%8A%A4%ED%83%80%EC%9D%BC)
  - WASD
  - `E` : 카메라를 위로
  - `Q` : 카메라를 아래로
  - `Z` : 줌아웃
  - `C` : 줌인

- `NavMeshBoundsVolume` 를 레벨에 배치 하고나서 `P`
  - 녹색으로 길찾기를 수행할 수 있는 영역 표시

- 오브젝트의 이동 축을 잡고 `End`
  - 오브젝트가 맵 바닥에 딱 붙는 위치로 이동됨

- `G`
  - 기즈모 끄기

- `Alt` 누른 상태로 해당 오브젝트의 이동 축을 잡고 드래그
  - 복사하기

- `Shift + F1`
  - 뷰포트에서 플레이 중에 마우스 커서 되살리기

- 플레이 중 `PageUp`, `PageDown`
  - 맵에서 현재 선택된(활성화된) Actor를 바꿀 수 있음

- 플레이 중 `F8`
  - 캐릭터에 빙의된 상태 해제