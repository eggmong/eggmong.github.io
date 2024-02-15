---
title:  "[Hamster is Free!] 개발일지 1-2"
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

# <Hamster is Free!> 개발일지 1-2

[전 글 (링크)](https://eggmong.github.io/devlog/HamsterisFree_01/#hamster-is-free-%EA%B0%9C%EB%B0%9C%EC%9D%BC%EC%A7%80-1)과 이어지는 글이다.  
이전 글에선 시스템적인 부분들을 작업한 내역을 간단하게 작성했고,  
이 글에선 Game 부분 작업 내역을 작성 해 볼 것이다.  

## Game 구성요소 - Player class

![log1]({{ site.imageurl }}UnityDocs/2023-09-08-HamsterisFree/2023-09-08-2_log1.png)  

<Hamster is Free!> 는 `한 줄 긋기` 게임이다.  
반드시 한 번에 드래그하여 ExitTile에 도달해야 스테이지가 클리어 되는 것이다.  
드래그를 하여 맵으로부터 길을 개척해 나아간다는 느낌으로 구현하고 싶었다.  
그래서 드래그를 할 때 `Line Renderer` 를 사용하여 드래그 궤적을 남겼다.  
(이 궤적이 길인 것 처럼 표현하고 싶었다. 지금은 그저 흰색이지만, 나중엔 텍스쳐를 넣어야지... )

![log2]({{ site.imageurl }}UnityDocs/2023-09-08-HamsterisFree/2023-09-08-2_log2.png)  

그리고 Mask.  
게임을 시작 하게되면 일정 시간(1초) 동안 꾹 누르고 있어야 게임이 정상적으로 시작되는데,  
그 1초동안 숨겨져있던 Cover 이미지가 FadeIn 으로 나타나 맵 위를 덮는다.  
그리고 유저의 터치 영역을 동그랗게 masking 하는데,  
이 마스킹된 구멍으로 플레이어는 맵을 살펴가며 아이템을 먹고, 장애물을 피해가며 탈출해야 하는 것이다.  

(지금은 기획적으로 고민중이다. 장애물까지 Cover 이미지로 덮어버리니, 난이도가 너무 높은 것 같다. ㅠㅠ)  

## Game 구성요소 - Tile

게임은 SeedTile, MonsterTile, ExitTile 3가지 종류로 구성된다.

`SeedTile`은 유저가 먹을 수 있는 아이템이다.  
ExitTile에 도달했을 때 먹은 SeedTile 갯수에 따라 별(보상)을 지급할 예정이다.  

`MonsterTile`은 유저가 피해야 하는 대상이다. 닿게 되면 처음부터 다시 드래그를 해야 한다.  

`ExitTile` 은 최종적으로 도달해야 할 타일이다. 여기에 닿으면 스테이지 클리어로 간주한다.  

세 타일 모두 공통적으로 사용하는 부분이 있어서
`TileBase` 라는 부모클래스를 각각 상속받아 구현하였다.  
그리고 `Builder` 패턴을 사용하여 SeedTile이나 MonsterTile에 필요한 속성을 추가적으로 붙여주었다.  

```c#
public abstract class TileBase : MonoBehaviour
{
    [Serializable]
    public struct TileInfo
    {
        public Define.TileType Type;
        public int RootIdx;

        public string SubType;
        public int SubTypeIndex;
        public float ActiveTime;
    }

    // Builder 패턴
    public class TileBuilder
    {
        private TileInfo _tileInfo = new TileInfo();

        public TileBuilder(TileInfo _Info)
        {
            _tileInfo = _Info;

            // 추가 정보 기본값으로 초기화
            _tileInfo.SubType = "";
            _tileInfo.SubTypeIndex = -1;
            _tileInfo.ActiveTime = 0f;
        }

        ...

        /// <summary>
        /// 각 타일들의 서브 타입 지정
        /// </summary>
        /// <param name="_Type">eg. SeedTile의 Default, Disappear, Fake 타입</param>
        public TileBuilder WithSubType(string _Type)
        {
            _tileInfo.SubType = _Type;
            return this;
        }
    }
```

### Game 구성요소 - Tile 의 Act

SeedTile 이나 MonsterTile은 각각 특별한 능력이 부여된다.  
일정 간격으로 나타났다 사라지거나, 시간이 지나면 아예 소멸되거나,  
위치가 바뀌는 능력 등이 있다.  

이걸 구현해주기 위해 `interface` 를 사용했다.  

```c#
public interface ITileActor
{
    public abstract UniTask<bool> Act(TileBase _Tile, CancellationTokenSource _Cts = default, float _ActiveTime = 0f);
}
```

각 타일들은 위의 인터페이스를 상속 받아 기능이 부여된다.  

```c#
switch (this.subType)
    {
        case Define.TileType_Sub.Disappear:
        {
            this.tileActor = new TileActor_Disappear();
        }
        ...
    }

...

var task = this.tileActor.Act(this, this.cts, this.info.ActiveTime);

```

`new TileActor_Disappear();` 로 기능을 넣어주고, `Act` 함수를 호출하여 기능이 실행되도록 했다.  

### Game 구성요소 - Tile 의 TileTrigger

모든 타일들은 반드시 Player와 충돌하는 Trigger 이벤트가 발생한다.  
(Seed는 유저가 드래그하다 먹을 수 있고, Monster는 닿으면 진행 리셋, Exit는 닿으면 스테이지 클리어)  

그래서 부모 클래스인 `TileBase` 에 TileTrigger 함수를 만들어주었다.
```c#
public abstract UniTaskVoid TileTrigger();
```
`abstract` 을 사용하여 모든 타일들은 Trigger 이벤트에 대한 내용을 반드시 구현하도록 했다.  


## Manager (GameManager, StageManager, MapManager)

각 Manager들은 간단히 설명하겠다.  

- GameManager
  - Game Scene을 총괄하는 매니저이다.  게임이 시작되고, 종료되었을 때의 플로우를 담당한다.  
  - StageManager, MapManager의 인스턴스를 들고 있어 다른 클래스에서 접근하기 용이하도록 했다.  

- StageManager
  - 이름대로 스테이지 처리를 하는 매니저이다. 
  - 데이터테이블에서 읽어온 대로 Stage Type, Stage Type에 따른 제한요소 처리 (Time 이나 TryCount) 를 한다.  
  - Game UI 중 TopBar 에 위치한 스테이지 정보를 업데이트 해준다.  

- MapManager
  - DataContainer 에 있는 Stage 데이터들로 Stage의 맵 세팅을 해주는 클래스이다.

## Shader

회사에서 shader를 조금 만져보게 된 이후 관심이 생겨서 공부중이다.  
개발 중인 게임에도 셰이더를 적용해보고 싶어서, 뭘 해볼지 고민하다가  
게임 배경이 조금 밋밋한 듯 하여 셰이더로 그라데이션 애니를 넣어주는 작업을 해보았다.  

<!--![shader]-->
<img src="{{ site.imageurl }}UnityDocs/2023-09-08-HamsterisFree/2023-09-08-2_log3.GIF">  

Unity의 Shader Graph로 만들었다.  

![shader2]({{ site.imageurl }}UnityDocs/2023-09-08-HamsterisFree/2023-09-08-2_log4.png)  

## Art Resources

틈틈히 리소스도 생산해봄 ㅋㅋㅋ
위에 보였던 모든 리소스들은 내가 직접 그린 것들이다.  

그리고 Intro 씬에 나올 로고도 그려보았다.

<!--![res]-->
<img src="{{ site.imageurl }}UnityDocs/2023-09-08-HamsterisFree/2023-09-08-2_log5.GIF">  

사실 이 프로젝트는 나 혼자 개발할 원맨팀이었는데. (물론 지금도 내가 다 작업하고 있다!)  
추후 붙일 백엔드 작업 및 빌드를 뽑기위한 CI/CD 작업을 해주시겠다는 분이 계셔서,  
함께 듀오를 결성하게 되었다.  
그리하여 슈크림팀. ㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋ
이름에 충실하게 슈크림을 본따서 그려보았다. 수정을 한 5번 했더니 꽤 맘에 드는 결과물이 나와서,  
Intro 뿐만 아니라 Github 의 조직 로고에도 넣었다.  


사운드도 찾아야 하는데 너무.. 너무 싫다!!! 귀찮다!!!!!!!! 일일히 들어야 하는게 고문 같아!



## 그리고 현재

현재는 이제 게임의 구성 요소 작업들을 마무리 짓고,  
게임이 한 사이클 돌 수 있도록 남은 작업들을 하고 있다.  

`JsonManager` 라는 클래스를 만들어 유저의 진행 상황을 저장하고  
스테이지 재진입 시 해당 스테이지 상태를 재로드해서 구성하는 작업을 하고 있다.  

유저가 갖고있는 보상(별) 도 저장해야겠지?  

그리고 GameManager 에서 StartFlow, EndFlow 를 구현해야 한다.  

그다음엔 Lobby Scene을 작업해야 한다.  

그다음엔 광고를 붙이고...  

이번 달에 끝내는 건 무리려나... (파스스)  
10월엔 끝내서 출시할 수 있을까? ㅠㅠ

그래도 화이팅~~~