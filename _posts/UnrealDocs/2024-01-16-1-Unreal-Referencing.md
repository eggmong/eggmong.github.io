---
title:  "소프트 레퍼런싱 vs 하드 레퍼런싱"

categories:
  - UnrealDocs
tags:
  - [Unreal, Game Engine]

toc: true
toc_sticky: true
 
date: 2024-01-16
last_modified_at: 2024-01-16
---

<br>

# 소프트 레퍼런싱 vs 하드 레퍼런싱

## TObjectPtr

Actor 로딩시 `TObjectPtr` 로 선언한 언리얼 오브젝트도 메모리에 로딩됨.  
이를 <b>하드 레퍼런싱</b>이라고 한다.  

```cpp
UPROPERTY(EditAnywhere, Category = Weapon)
TObjectPtr<USkeletalMesh> WeaponMesh;

// ... 

void AABCharacterBase::EquipWeapon(UABItemData* InItemData)
{
    UE_LOG(LogABCharacter, Log, TEXT("Equip Weapon"));
    
    UABWeaponItemData* WeaponItemData = Cast<UABWeaponItemData>(InItemData);

    if (WeaponItemData)
    {
        Weapon->SetSkeletalMesh(WeaponItemData->WeaponMesh);
    }
}
```

게임 진행에 필수적인 언리얼 오브젝트는 이렇게 같이 로딩되어도 되지만  
아이템의 경우엔... 데이터 라이브러리에 1000종의 아이템 목록이 있을 경우  
다 로딩하는 건 메모리에 부담스럽다.  

> 프로젝트 실행하고,  
> ->틸트 (``)<- 키를 누르고,
> `Obj List Class=SkeletalMesh` 를 입력하면  
> 현재 로드된 SkeletalMesh를 볼 수 있다.  

![load]({{ site.imageurl }}UnrealDocs/UnrealUObjectPtrLoad.png)  
![load]({{ site.imageurl }}UnrealDocs/UnrealUObjectPtrLoad2.png)  

아직 박스 안의 아이템을 먹지도 않았는데  
코드 상에서 로드를 했기 때문에  
DragonSword를 로드하고 있었음.  


## TSoftObjectPtr


그래서 우리가 필요한 데이터만 로딩하도록  
멤버변수를 선언하되, 로딩하지 않게  `TSoftObjectPtr` 로 선언해주면  
에셋 대신에 <b>에셋 주소 문자열</b>이 지정이 되면서  
우리가 필요한 때 에셋을 로딩할 수 있다.  



```cpp
UPROPERTY(EditAnywhere, Category = Weapon)
TSoftObjectPtr<USkeletalMesh> WeaponMesh;

// ...

// 변수가 할당 됐는지 안됐는지 모르기 때문에 바로 사용 할 수 없음
void AABCharacterBase::EquipWeapon(UABItemData* InItemData)
{
    UE_LOG(LogABCharacter, Log, TEXT("Equip Weapon"));
    
    UABWeaponItemData* WeaponItemData = Cast<UABWeaponItemData>(InItemData);

    if (WeaponItemData)
    {
        if (WeaponItemData->WeaponMesh.IsPending())         // 아직 로딩이 안되어 있다면
        {
            WeaponItemData->WeaponMesh.LoadSynchronous();   // 동기적으로 로딩하도록 함
        }

        Weapon->SetSkeletalMesh(WeaponItemData->WeaponMesh.Get());  // TSoftObjectPtr 로 로드한 건 .Get() 으로 가져올 수 있음
    }
}
```