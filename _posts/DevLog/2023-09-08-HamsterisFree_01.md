---
title:  "[Hamster is Free!] 개발일지 1-1"
excerpt: "7월 27일 ~ 9월 8일 (44일) 까지의 기록"

categories:
  - DevLog
tags:
  - [DevLog]

toc: true
toc_sticky: true
 
date: 2023-09-08
last_modified_at: 2023-09-08
comments: true
---

<br>

# <Hamster is Free!> 개발일지 1-1

작업한 것들을 나중에 회상할 수 있도록 기록을 남겨두는게 좋을 것 같아서 개발일지를 쓰기로 했다.  
첫 커밋부터 현재(23.09.08) 까지의 작업 내역을 기록해두겠다.  

프로젝트를 시작하게 된 배경은... 이젠 나도 짬도 찼으니(?) `1인 개발로 게임을 출시해보고 싶다!`  
기왕이면 Github에 Repository를 만들어서 기록도 하고,  
스토어에 올리는 것 까지 전부 혼자 해보고 싶다는 생각이 들어서 시작하게 되었다.  

그래서 내가 좋아하는 `햄스터` 를 캐릭터로 삼고,  
첫 1인 개발 게임이니까 최대한 간단하게 (작업적으론 그리 간단하지 않았지만 ㅠ) 심플한 게임으로 만들어보자고 생각했다.  

- `기획`은 간단하면서도 어설프지 않도록  
- 스토어에 올리기로 결심한 이상 `아트 퀄리티`도 챙기기 (프로그래머가 할 수 있는 최선으로 ㅠ)  
- 프로그래머인 이상 `코드`에 공을 들여보자!  

이렇게 원칙으로 삼고 시작하게 되었다.  

![log1](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-09-08_log1.png)  

찾아보니 첫 커밋 날짜가 7월 27일이었당.  

작업을 시작하고 8월 한 달 동안은 정말 미친듯이 작업에 몰두했던 것 같다.  
퇴근하고 저녁을 먹으면 8시쯤이었는데, 그 때 부터 새벽까지 매일매일 작업하고, 주말에도 작업했으니까...  
나의 빽빽한 깃허브 잔디들이 증명해주고 있다.  

![log2](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-09-08_log2.png)    


처음 시스템 베이스부터 전부 스스로 구현해보려고 하다보니 시간도 오래 걸렸지만,  
회사에서 주어진 일을 할 때보다 더 많은 걸 배울 수 있었다.  

## DataTable (SheetDownloader)

우선 게임에 필요한 `데이터테이블` 작업부터 소개하자면.  
나의 집 컴퓨터엔 엑셀이 깔려있지 않으므로 `Google Sheets` 를 사용하기로 했다.  
처음 작업 내용은 이랬다.  

1. 시트에서 내용을 기입하고, 
2. csv로 직접 다운받아 Unity Editor에 집어넣고, 
3. csv 파일을 파싱하여 데이터들을 읽어오는 클래스를 구현

하지만... 내 손으로 직.접. 사이트에 들어가 다운 받는게 귀찮았고!  
`ScriptableObject` 를 사용하여 데이터들에 접근하자! 라는 생각이 들었다.
그리하여 현재는 이런 방식이다.  


![log3](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-09-08_log3.png)    
![log4](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-09-08_log4.png)    
![log5](https://github.com/eggmong/eggmongImages/raw/main/UnityDocs/2023-09-08_log5.png)    

1. 버튼 하나를 눌러 Editor 상에서 Google Sheets들을 `csv로 다운`
2. 다운 받은 `csv들을 파싱`하여 각각의 데이터클래스 기반으로 만들어진 `ScriptableObject로 자동 생성`
3. `DataContainer` 라는 Prefab에 `ScriptableObject 자동 링크`

작업물은 [Repository(링크)](https://github.com/SukereamTeam/hamsterisfree) 에서 확인할 수 있지만...  
지금은 private 상태이다. (23.09.08)  
마켓에 업로드 한 후 레포지토리를 공개할 생각이다.

## Scene Controller

제일 우여곡절이 많았던 작업이다.  
SceneController를 직접 구현해보는 게 어려웠다.  

### Scene 전환 - Task

`UniTask` 와 `UniRx` 를 사용하여 구현했고,  
다음 씬으로 완전히 전환되기 전에 `Task`로 미리 리소스 로딩을 해주고 싶었다.  

```c#
private List<UniTask> loadingTask;
    public List<UniTask> LoadingTask
    {
        get => this.loadingTask;
        private set => this.loadingTask = value;
    }
```

다음 씬으로 완전히 전환되기 전에 loadingTask 에 해야 할 작업들을 List로 Add해둔다.

```c#
await UniTask.WhenAll(LoadingTask.ToArray().Select(async task =>
    {
        await task;

        if(loadingScene)
        {
            CompleteCount++;
            var amount = ((float)CompleteCount / (float)TaskCount);
            var progress = Mathf.Round(amount * 100) / 100;    // 소수점 둘째자리까지 반올림
            loadingScene.UpdateProgress(progress);
        }
    }));
```

loadingScene 을 사용할 경우 로딩 바에 업데이트를 해주고,
그게 아니라면 task 가 끝날 때 까지 기다린다.

### Scene 전환 - Fade

SceneController에 `Fade` 기능도 넣어서 씬 전환 사이에 FadeIn, FadeOut 기능을 구현했다.  

```c#
public async UniTask Fade(bool _FadeIn, float _Duration, bool _Skip, CancellationTokenSource _Cts, Action _Action = null)
    {
        var fadeInt = _FadeIn ? 1 : 0;  // fade in : 검은 화면에서 서서히 밝아지는 것!
        this.fade.color = new Color(this.fade.color.r, this.fade.color.g, this.fade.color.b, fadeInt);
        this.fade.raycastTarget = !_Skip;
        
        ...

        using (cancellationToken.Token.Register(() => cancelAction()))
            {
                await tweener
                    .OnKill(() =>
                    {
                        Debug.Log("Fade was OnKill.");

                        this.fade.raycastTarget = false;
                    })
                    .OnComplete(() =>
                    {
                        Debug.Log("Fade was OnComplete.");
                    }).ToUniTask();
            }
    }
```

bool 변수로 _Skip 값을 받아서, 스킵할 수 있는 경우 (예를 들어 Intro 씬에서의 Fade.)
화면을 터치하여 Fade 작업을 취소시키도록 했다.

이 역시 자세한 내역은 [Repository(링크)](https://github.com/SukereamTeam/hamsterisfree) 에서 확인할 수 있지만...  
현재(23.09.08) 는 비공개 상태이다.

## DataContainer

데이터테이블로 사용하는 `ScriptableObject` 를 들고 있는 싱글톤 클래스이다.  

뿐만 아니라 스테이지마다 사용할 Sprite들도 들고 있다.  

1. Lobby에서 스테이지 선택
2. Game 씬으로 전환되기 전 DataContainer에서 StageTable 을 읽어와 해당 Stage에 필요한 리소스 체크
3. 리소스 로드하여 `Dictionary` 로 담아두기
4. Game 씬으로 전환된 후 DataContainer에 담긴 Sprite 들을 기반으로 맵 세팅

이런식으로 구현했다.  


## Singleton

Manager 기능을 하는 클래스들은 싱글톤으로 구현해두는 게 아무래도 접근하기 쉬워서 선택했다.  
기본적인 `Singleton` 이 있고, MonoBehaviour 을 상속받는 `MonoSingleton` 이렇게 두 가지로 나누어 만들어 사용하고 있다.  

MonoSingleton 도 2가지로 나뉘는데, `DontDestroyOnLoad` 를 사용하는 `GlobalMonoSingleton` 이 있고  
사용하지 않는 기본 `MonoSingleton` 클래스가 있다.  
게임의 전체를 총괄하는 CommonManager 는 GlobalMonoSingleton 을 상속받아 사용중이고,  
게임의 Stage를 담당하는 GameManager 는 MonoSingleton 을 상속받아 사용한다.  

## CameraResolution

해상도 대응을 해주기 위해 CameraResolution 라는 클래스를 만들어 카메라에 붙여주었다.  
다른 씬들은 대부분 UGUI 로 구성되어서 Canvas에서 알아서 세팅하면 되지만,  
게임 씬의 경우 UI 가 아닌 World Space에 올라간 오브젝트들이라  
게임 화면이 잘리면 안되므로 필요한 작업이었다.  

여기서도 우여곡절이 하나 있었는데, 처음엔 나름 간지나게(...) 짜본다고 휘황찬란하게;;  
무슨 비율을 곱해주고 어쩌고 했다가  
나중에 빌드 뽑아보니 화면이 뿌옇게 나와서... 잘못된 코드라는 걸 깨닫고 심플하게 바꿨다.  

기기 해상도 비율이 내가 타겟으로 지정해놓은 비율을 벗어나면 (난 9:16으로 했다)  
mainCamera.orthographicSize 에 값을 보정해주어 조절하도록 했다.  



시스템 작업 내역만 작성했는데 좀 길어지는 것 같아서  
Game 작업 내역은 개발일지 1-2로 나누어 작성하기로 했다.  
기록을 남기는 건 좋은 것 같다.  
대략 지금까지 1달 2주 쯤 작업했고, 나는 별로 한 게 없다고 생각했는데  
적다보니 꽤나 했구나 싶다.  