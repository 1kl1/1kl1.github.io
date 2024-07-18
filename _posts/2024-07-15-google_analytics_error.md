---
title: Missing google_app_id. Firebase Analytics disabled.
date: 2024-07-15 00:00:00
categories: [개발, flutter, android]
tags: [개발, flutter, android, firebase, flutterfire]
description: Flutter에서 Missing google_app_id. Firebase Analytics disabled.오류 발생 시 해결방법



---



## Missing google_app_id. Firebase Analytics disabled. [ANDROID]

flutterfire로 firebase를 안드로이드 프로젝트에서 설정한 뒤 analytics를 적용 시

해당 로그와 함께 analytics가 초기화가 되지 않는 오류가 있습니다.

구버전의 flutterfire cli와 새로운 버전의 gradle과 flutter sdk를 사용 시 해당 오류가 발생하는 것으로 추정됩니다.

다음과 같은 방법으로 해결했습니다.

`dart pub global activate flutterfire_cli` > flutterfire cli 업데이트

`flutter clean`

`flutter pub get`

`flutterfire configure`

```
// android>app>build.gradle

plugins{
	...
	id "com.google.gms.google-services" // 추가
}

```

\* android>build.gradle 파일은 수정하지 않았습니다.

문제 해결을 담은 많은 블로그와 stack overflow에선 루트의 build.gradle과 app의 build.gradle

둘 다 수정하는 방법으로 해결했으나 제 환경에선 작동하지 않았습니다.

