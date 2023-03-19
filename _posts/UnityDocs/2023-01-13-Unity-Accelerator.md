---
title:  "[Unity] Accelerator (캐시서버) 구축"

categories:
  - UnityDocs
tags:
  - [Unity, Game Engine]

toc: false
toc_sticky: false
 
date: 2023-01-13
last_modified_at: 2023-01-15
---

# Unity Accelerator

## Unity 공식 문서

>[https://docs.unity3d.com/kr/2020.3/Manual/UnityAccelerator.html#install](https://docs.unity3d.com/kr/2020.3/Manual/UnityAccelerator.html#install)

## 설치 및 사용방법

<p>

- Accelerator 설치 방법

    Unity Teams 구독을 하지 않았다면 [Unity Accelerator 문서](https://docs.unity3d.com/kr/2020.3/Manual/UnityAccelerator.html#install)에서 설치 프로그램을 다운 받을 수 있다.

    작업자의 컴퓨터가 아닌 <u>헤드리스 컴퓨터</u>에 설치한다. (빌드머신 등)

    설치를 진행하다보면 작업자들이 참조할 수 있는 IP주소가 나타난다. 이것을 작업자들의 Unity 에디터에서 입력하면 된다.

- 에디터에서 Accelerator 사용하기

    Unity 에디터에서 Edit → Project Setting 선택 후 위에서 받은 IP주소를 입력하면 된다.
</p>

<br>


```
💡 엑셀레이터(캐시서버)를 사용하는 이유 : 에셋 임포트 시간 단축!
```

<br>



## Asset Importing 과정

- 어떤 파일을 임포트하면 런타임에서 사용할 수 있는 포맷으로 변환하고, 유니티는 이 파일을 GPU에 전달해 런타임 중에 실시간으로 표시되게 한다.
    
- 파일의 정보는 바뀌지 않고, 처리와 변환을 거친 데이터 버전은 라이브러리 폴더의 에셋 캐시에 저장된다.
    
- Unity는 소스 에셋 파일과 에셋에 종속되는 데이터에서 임포트한 버전의 에셋을 다시 생성할 수 있다. 그래서 라이브러리 폴더에 변환된 버전의 에셋이 있음.

> <b>Asset 파일이 많아지거나, 플랫폼을 변경한다면 임포트 하는데에 많은 시간이 소요될 수 있다.</b>

변경된 Asset 파일을 팀원들이 확인한다고 한다면,

1. VCS에서 변경된 Asset을 가져옴
2. 프로젝트를 엶
3. 변경된 모든 Asset이 임포트 되고 업데이트 될 때 까지 기다림

을 지나야 확인할 수 있다.

<p>

> 이를 해결하기 위한 것이 <b>엑셀러레이터(캐시서버)</b>인 것이다.

</p>

<br>

## Accelerator의 특징

<p>

- 로컬 네트워크 프록시 및 캐시 서비스
- 작업하는 팀원 간의 에셋 공유 조정
- 로컬 네트워크에 설치되면 Unity Editor는 엑셀러레이터를 통해 다른 팀원이 요청, 변경하거나 빌드한 에셋을 저장하고 검색함
- 임포트한 버전의 에셋 복사본을 보관해, 다시 임포트하는 데 소요되는 시간 단축함
    * <mark style='background-color: #bae7af'>즉 제일 처음 사람만 리임포트 하고, 그 결과는 Accelerator에 자동으로 캐싱됨</mark>
    * 다음에 다른 팀원이 동일한 버전의 에셋을 임포트 할 땐, 로컬 컴퓨터에서 임포트 프로세스를 시작하기 전에 캐시부터 확인하게 됨.

</p>

<br>

### Accelerator의 장점
- <mark style='background-color: #f5f0ff'>Accelerator가 없는 경우🤔</mark>
    * 동일한 에셋에 대한 리임포트를
    각자 모두 시작하므로 시간이 허비됨

<br>

- <mark style='background-color: #fff5b1'>Accelerator가 있는 경우😊</mark>
    1. 사용자 A가 에셋을 변경하고, 변경된 내용을 리임포트 함
    2. 자동으로 캐싱됨
    3. 사용자 A가 에셋을 VCS로 push함
    4. 다른 팀원 B가 VCS에서 내려받으면, 변경된 에셋을 재임포트 하는게 아니라 Accelerator에서 가져온 임포트된 버전의 에셋을 다운로드 한다.

<br>

<details>
<summary>Accelerator vs Cash server (Accelerator 출시 전의 cash server를 의미)</summary>
<div markdown="1">       

- 사용하지 않는 에셋 데이터를 자동으로 정리해줌.
- 커스텀 스크립트로 캐시 관리를 할 수 있음
- 대시보드가 있어서 지표 및 로그 확인 가능 (그 외 CPU 사용률 등 확인 가능)
- 다중 인스턴스 미러링 → 캐시 데이터를 다른 Accelerator 와 동기화 할 수 있음

</div>
</details>

***