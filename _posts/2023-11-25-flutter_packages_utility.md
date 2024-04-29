---
title: Flutter에서 유용하게 사용했던 패키지 Utility 편
date: 2023-11-25 00:00:00
categories: [개발, flutter]
tags: [개발, flutter]
description: intl, go_router, permission_handler, freezed, shimmer


---

Flutter의 오픈소스 커뮤니티를 이용해서 이제껏 정말 빠르고 쉽게 개발을 해 왔습니다.

구글에서도 적극적으로 밀고있는 https://pub.dev/ 입니다.

저 또한 잘 사용해 왔었는데, 문득 flutter로 개발해 온 2년간 유용하게 사용해왔던 패키지들을

정리해 놓으면 다른 이들에게 도움이 될 수 있어 좋겠다는 생각이 들어 정리 해 보았습니다.

utility편 입니다.



## intl

요약: 사용하는 언어에 맞게 date/number formatting, message translation, bidrectional text를 제공 해 주는 유용한 패키지

종종 숫자를 formatting 해야 할 때가 있습니다.

예를 들어 10000000이라는 숫자를 생각 해 보겠습니다.

1000만인데, 이 숫자는 10,000,000으로 써야 가독성이 좋습니다.

또는 이 숫자가 돈을 나타낸다고 할 때 10,000,000원 이라고 쓰여야 유저 입장에서 편하게 읽을 수 있을 것 입니다.

```dart
NumberFormat("###,###,###,###,###원").format([number])
```

이 코드 한줄로 format이 가능합니다.



어떨 땐 어플리케이션이 한국만을 타겟하는 게 아닐 때도 있습니다.

그럴 땐 '원'이 아니라 'usd' 혹은 'VND' 등 이 될 수도 있는데, 이럴 때 손쉽게 localization을 할 수 있는 기능을 제공 해 줍니다.

```dart
initializeDateFormatting('ko_KR', null).then(
  (_) async => runApp(
    ...
  ),
)
```



숫자 뿐만이 아니라 날짜도 마찬가지입니다.

dart에서 제공하는 강력한 클래스 DateTime이 있지만 이를 적재적소에 맞게 formatting 하는 건 다른 일입니다.

2023-11-24라는 날짜가 있다면, 이 날짜를 2023년 11월 24일로, 혹은 20231124, 2023/11/24 등 여러가지 포멧으로 표현 가능합니다.

```dart
DateFormat('yyyy년 M월 d일').format([datetime]);
DateFormat('yyyyMd').format([datetime]);;
DateFormat('yyyy/M/d').format([datetime])
```

모든 프로젝트에서 유용하게 사용할 수 있는 패키지입니다.

## go_router

## permission_handler

## freezed









