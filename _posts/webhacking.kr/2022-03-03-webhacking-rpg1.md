---
title: "[webhacking.kr] Challenge RPG1 문제 풀이"
categories:
  - webhacking
tags:
  - webhacking.kr
  - wargame
  - RPG1
  - rpg
toc: true
toc_sticky: true
toc_label: "RPG1 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/webhacking.jpg
---

💡 Webhacking.kr challenge RPG1 문제에 대한 풀이입니다.
{: .notice--warning}

# 문제
![image](https://user-images.githubusercontent.com/33647663/156581827-4bcd2c39-f16d-4f62-828a-2d99075a75a0.png)


게임을 시작하면 다음과 같이 화면이 나오며, 화면에 강을 건너서 보물상자를 획득하면 클리어 하는 문제입니다.


# 문제 풀이
이러한 종류의 문제를 풀기위한 방법은 여러개가 있을 것입니다. 게임 로직을 분석해서, 맵에서 강이 있으면 이동하지 못하게 하는 로직이 있을 것이므로 해당 로직을 바꾸는 방법이나, 맵 자체에서 강을 지우거나 혹은 캐릭터의 좌표를 그냥 임의로 바꾸는 것 등 여러 방법이 있습니다.

저는 가장 쉬워보이는 캐릭터의 좌표를 찾아서 바꾸는 방법으로 접근해 보았습니다.

우선 게임의 동작은 js코드로 작동합니다. js폴더및의 javascript 코드들을 보면 다음과 같이 구성되어 있습니다.

![image](https://user-images.githubusercontent.com/33647663/156584027-b02800f1-c5ec-49ca-85c0-29a09d02fc78.png)

파일들의 코드를 쭉 훑어보면, rpg_manager.js 파일에서 게임의 전체 흐름을 처리하고 있음을 알 수 있습니다.

다음은 rpg_manager.js 파일에 있는 코드입니다.

```javascript
// line 200
DataManager.createGameObjects = function() {
    $gameTemp          = new Game_Temp();
    $gameSystem        = new Game_System();
    $gameScreen        = new Game_Screen();
    $gameTimer         = new Game_Timer();
    $gameMessage       = new Game_Message();
    $gameSwitches      = new Game_Switches();
    $gameVariables     = new Game_Variables();
    $gameSelfSwitches  = new Game_SelfSwitches();
    $gameActors        = new Game_Actors();
    $gameParty         = new Game_Party();
    $gameTroop         = new Game_Troop();
    $gameMap           = new Game_Map();
    $gamePlayer        = new Game_Player();
};

DataManager.setupNewGame = function() {
    this.createGameObjects();
    this.selectSavefileForNewGame();
    $gameParty.setupStartingMembers();
    $gamePlayer.reserveTransfer($dataSystem.startMapId,
        $dataSystem.startX, $dataSystem.startY);
    Graphics.frameCount = 0;
};
```

게임을 시작할때, DataManager.setupNewGame()함수가 동작하여, 여러가지 클래스들의 객체를 생성함을 알 수 있습니다. 즉 게임 생성시에 클래스의 인스턴스 값들이 저장되는 변수명을 알아냈습니다.

변수명을 보고 추측을 해보면, 게임 캐릭터의 객체는 `$gameActors` 나 `$gamePlayer`에 있을 것으로 생각되어서 두 객체를 모두 본 결과 `$gamePlayer`가 실제로 게임 케릭터의 정보를 가지고 있는 인스턴스였습니다.

위의 내용을 확인하는 방법은 크롬에서 `F12 > 소스 > 감시`에서 `$gamePlayer` 변수를 입력하여 값을 확인 가능합니다. 또한 `F12 > 콘솔 `에서 확인하는 방법도 있습니다.

![image](https://user-images.githubusercontent.com/33647663/156585256-200dcb3e-37a7-4931-969e-da17f286ba09.png){:. align-center}

![image](https://user-images.githubusercontent.com/33647663/156593753-4d23c57f-752c-439d-bc57-b4a725be9dfa.png){:. align-center}

상당히 많은 변수가 있는데, 이것들 중 좌표를 의미하는 _x,_y 변수가 있습니다.

![image](https://user-images.githubusercontent.com/33647663/156585572-59724b8b-9a5b-4fcc-9461-ccc1f848c243.png)

이 값을 수정하여 1,1로 만들면 다음과 같이 강을 넘어서 1,1의 좌표로 이동됩니다.

![image](https://user-images.githubusercontent.com/33647663/156586437-b0a993c8-dc54-46fd-94c3-429f7a518a63.png)

이제 이동해서 보물을 열면 플래그가 나옵니다.

![image](https://user-images.githubusercontent.com/33647663/156586584-9230d184-380f-450b-b5b9-284bb999aa0a.png)


# 추가로 해본 뻘짓.
## 비행기 타기

게임에서 여러가지 클래스들을 보니까 이 게임에서 보트나 비행기 등을 타는 기능이 있었습니다. 

```javascript
$gamePlayer._vehicleType="airship"
$gamePlayer._vehicleGettingOn= true
```
이렇게 설정하면, 비행기를 타는 모드가 되서 강을 그냥 건너갑니다.

![image](https://user-images.githubusercontent.com/33647663/156587937-2341a017-4521-4c0b-824d-2fd27562c562.png)

강을 건너고 나서는 다음의 설정을 해줘야 보물상자가 클릭이 됩니다.
```javascript
$gamePlayer._vehicleType="walk";
$gamePlayer._vehicleGettingOn=false;
$gamePlayer._transparent=false;
```


![image](https://user-images.githubusercontent.com/33647663/156588757-785f68b5-8e6b-479a-8372-68630b50dc18.png)


## 상자이동시키기
맵의 정보를 저장하고 있는 $gameMap을 변경시켜서 상자를 이동시킬 수도 있습니다. 하지만 상자를 이동시켜서 클릭을 하면, 플래그가 나오지 않습니다. 아마 내부적으로 초기 상자의 위치를 확인하는 로직이 있는 것 같습니다. 

![image](https://user-images.githubusercontent.com/33647663/156594219-965c10ea-df6e-4c41-8416-3117b1f755f4.png)

```javascript
$gameMap._events[1]._x=8
$gameMap._events[1]._y=8
```