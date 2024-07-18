---
title: Flutter에서 유용하게 사용했던 패키지 Utility 1편
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

routing을 손쉽게 할 수 있게 도와주는 패키지입니다.

기본적으로 build context에 extension도 제공해주기 때문에 직관적으로 사용할 수 있습니다.

deep linking도 지원하기때문에 거의 모든 프로젝트에서 사용하고 있습니다.

원래는 별도의 개발 팀에서 관리되고있는 것으로 알고 있었으나 어느 순간부터 flutter.dev가 관리하기 시작했습니다.



```dart
MaterialApp.router(
	...
  routeInformationParser: goRouter.routeInformationParser,
  routeInformationProvider: goRouter.routeInformationProvider,
  routerDelegate: goRouter.routerDelegate,
  ...
);
```

`MaterialApp` 에 go router를 전달해주면 사용할 수 있게 됩니다.

```dart
GoRouter(
  initialLocation: "/",
  refreshListenable: GoRouterRefreshStream(
    AuthService.to.isLoggedInObs.stream,
  ),
  redirect: (context, state) {
    final path = state.uri.toString();
    final isOnboard = path.startsWith("/onboard");
    if (!isOnboard) {
      if (AuthService.to.isLoggedIn) {
        return null;
      }
      return "/onboard";
    }
    return null;
  },
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomePage(),
    ),
  ],
  errorBuilder: (context, state) {
    return const ErrorPage();
  },
);
```

실제로 제가 개발중인 앱의 코드의 일부분입니다.

RefreshListenable은 listenable객체를 받고, 그 객체가 notifyListeners()를 호출했을 때 routing을 refresh합니다.

auth 상태에 따라서 routing이 바뀌어야 할때 유용합니다.

redirect와 routes, errorBuilder는 말 그대로 입니다.

`GoRouterRefreshStream`코드는 아래 첨부했습니다.

```dart

class GoRouterRefreshStream extends ChangeNotifier {
  GoRouterRefreshStream(Stream<dynamic> stream) {
    notifyListeners();
    _subscription = stream.asBroadcastStream().listen(
          (dynamic _) => notifyListeners(),
        );
  }

  late final StreamSubscription<dynamic> _subscription;

  @override
  void dispose() {
    _subscription.cancel();
    super.dispose();
  }
}

```



## permission_handler

android, ios를 개발 할 때에 permission이 필요한 기능들이 몇 있습니다.

제일 유명한 예시로 image picker를 사용할 때입니다.

이때 permission handler를 사용하면 permission의 상태와 함께 승인을 요청할 수 있습니다.

제 경우에는 permission이 필요한 기능 직전에 다음 함수를 호출해서 사용하고있습니다.

```dart
Future<bool> _checkPermission(Permission permission, String label) async {
  final status = await permission.status;
  switch (status) {
    case PermissionStatus.granted:
    case PermissionStatus.limited:
      return true;
    case PermissionStatus.denied:
      final result = await permission.request();
      if (result.isGranted) return true;
      SimpleNotify().show("Permission needed to this function work.");
      return false;
    case PermissionStatus.permanentlyDenied:
      openAppSettings();
      return false;
    default:
      return false;
  }
}
```

물론 이외에도 native한 설정이 필요하므로 패키지 페이지에서 참고해서 설정해주어야 합니다.



## freezed

data class들을 생성할 때 가장 유용하게 쓰고있는 패키지입니다.

vscode 사용 시 `Freezed` extension과 함께 사용하면 바로 폴더에서 우클릭 후 freezed클래스를 생성하기 위한 기본 파일을 생성해주기 때문에 사용하는것을 추천합니다.

`build_runner` 와`freezed_annotation`는 필수로 사용해야하고,  `json_serializable`을 이용하면 toJson, fromJson도 동시에 구현 할 수 있습니다.

- define a constructor + properties
- override `toString`, `operator ==`, `hashCode`
- implement a `copyWith` method to clone the object
- handle (de)serialization

기본적으로 이와 같은 기능을 구현해주기 때문에, immutable한 객체를 만드는게 훨씬 쉬워집니다.





