---
title: 2년차 flutter 주니어가 record를 쓰는 법
date: 2023-11-07 06:00:00
categories: [개발, flutter]
tags: [개발, flutter, dart3, records]
description: flutter, dart3에서 record를 쓰는 노하우와 꿀팁을 담은 글.
---

dart 3.0이 나오면서 `record`라는 새로운 타입을 소개했습니다.

저는 `record`는 기존에 있는 클래스, 타입들을 한 묶음으로 만들어서 쓸 수 있는 타입이라고 이해했습니다.

이 글에서는 어떻게 제가 실제 프로젝트에서 `record`를 사용했는지 소개해보겠습니다.



### database 구조가 결정되기 전 개발하기

저는 현재 다니고 있는 첫 회사가 스타트업입니다. 그래서, 일반적인 회사에서 걸치는 개발 절차? 관례에 대해서 전혀 모르고 실제 프로덕트 개발을 시작하게 되었습니다.

서버 개발자가 1명, 프론트엔드 개발자가 1명인데 서버의 자원이 필요한 기능이 있을 때, 서버가 먼저 개발을 마치고 실제 data를 준다고 하면 문제가 없지만 실제 상황은 그렇게 녹록치 않고, 두 개발을 동시에 진행하게 되었습니다.

이 때 api response를 서버에서 어떻게 줄 지 결정되지 않은 상태로 개발을 할 떄 record를 사용했습니다.

```dart
const HomeVideoItem({Key? key, required this.video}) : super(key: key);

final ({Video video, String channelName, String channelThumbnail}) video;
```

위의 코드에서 HomeVideoItem은 실제 Widget입니다. channel 정보와 Video 정보를 받아서 UI로 빌드 해 주는 widget입니다.

이 경우 서버에서 channel + video 정보를 어떻게 줄 지 결정되지 않은 상태에서, sample data를 만들고,

```dart
(
  video: video,
  channelName: channel.channel_name,
  channelThumbnail: channel.thumbnail_url,
)
```

해당 record를 만들어서 개발했습니다. record를 사용하지 않는다고 하면 하나의 변수를 이용하지 못하고 여러개의 변수를 이용하여 나중에 변경이 귀찮아지는 면이 있다면, 추후 개발이 완료되면 해당 record만 변경 해 주면 되기 때문에 용이했던 면이 있습니다.



### 여러개의 타입을 리턴하는 함수 구현

예컨데 Channel이라는 클래스가 있고, 그 클래스 안에 type이라는 필드가 있다고 가정 해 보겠습니다.

Channel List를 보여주어야 할 때, 동시에 type끼리 따로 모아서 별도로 list를 구성해야 하는 UI가 있다고 한다면(생각 이상으로 한 클래스 안에서 분화되어 보여주는 경우가 많았습니다.)

보통 이런식으로 구현하게 될 것입니다.

```dart 

List<Channel> channels = getChannels(); // List<Channel>을 return 합니다.


Column(
	children:[
    ChannelAList(
    	channels: channels.where((e)=>e.type == ChannelTypeA),
    ),
    ChannelBList(
    	channels: channels.where((e)=>e.type == ChannelTypeB),
    ),
    ...
  ],
)

```

저는 이렇게 UI 상에 where 함수가 쓰여 가독성을 해치는점, 추후에 변경할 때 용이하지 않을 것 같다는 생각이 들어 다음과 같이 변경했습니다.

```dart
// ({List<Channel> A, List<Channel> B}) 를 return합니다
({List<Channel> A, List<Channel> B}) channels = getSortedChannels(); 


Column(
	children:[
    ChannelAList(
    	channels: channels.A,
    ),
    ChannelBList(
    	channels: channels.B,
    ),
    ...
  ],
)

```

훨씬 가독성이 나아 보였습니다.



### class를 만들기엔 애매하지만 연관성이 있는 변수 묶기

앱 내 로그인을 구현할때 항상 따라오는 두 변수가 있습니다.

`id`와 `pw`입니다. 

변수가 2개여서 클래스를 만들긴 애매하지만 분명히 묶어서 사용되는 변수들이라고 볼 수 있습니다.

UI에서 입력 받아서 내부 로직을 거치고 서버에 data를 보내는 함수에까지 전달을 해 주어야 하는 친구들이죠.

이럴 때 두 변수를 묶어서 record로 사용하면 편했습니다.

```dart
String? id;
String? password;

// 위 코드를 아래와 같이 리펙토링 했습니다.
({String id, String password})? loginInfo;

// 위의 변수는 다음과 같이 참조 가능합니다
loginInfo.id;
loginInfo.password;
```

이처럼 사용하면 좀 더 관리하기 편한 면이 있었습니다.

---

추후 이 외 용도로 record를 사용하게 되면 더 추가하도록 하겠습니다.

댓글로 본인만의 record 사용 꿀팁을 소개해주세요!
