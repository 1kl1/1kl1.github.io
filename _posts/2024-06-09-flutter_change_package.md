---
title: Flutter에서 앱 패키지 이름 변경법
date: 2024-06-09 00:00:00
categories: [개발, flutter]
tags: [개발, flutter, android, ios, app package, packagename]
description: Flutter에서 앱 패키지 변경방법, ios, android 앱 패키지 변경법. 앱 패키지 이름 변경 중 에러 발생할때


---



## Flutter에서 앱 패키지 이름 변경법

### Android

1. `/android/app/build.gradle`파일에서 android > defaultConfig 내의 `applicationId`를 수정합니다.

   통일성을 위해서 namespace도 수정 가능합니다.

   namespace를 수정 시 android>app>src>main>kotlin> ... 하위에 있는 `MainActivity.kt`파일에서 packge를 수정 해 주면 됩니다.

```dart
// /android/app/build.gradle

...
android {
  	namespace ...
    defaultConfig {
        applicationId "com.example.myapp"
        minSdkVersion 15
        targetSdkVersion 24
        versionCode 1
        versionName "1.0"
    }
    ...
}
...
```

2. `/app/src/{debug|main|profile}/AndroidManifest.xml` 총 3가지 위치에 있는 `AndroidManifest.xml`  파일에서 `<manifest>` 태그의 옵션값인 `package`를 찾아 변경해줍니다. 

   없을 경우 생략 가능합니다. 제 경우에는 생략했습니다.

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.example.myapp">
    ...
</manifest>

```



### Ios

Xcode로 `ios/Runner.xcworkspace`를 열어줍니다.

`Runner > TARGETS Runner > Bundle Identifier` 항목을 수정해 주면 됩니다.

![Screenshot 2024-06-09 at 10.07.32 AM](../assets/img/2024-06-09-flutter_change_package/Screenshot%202024-06-09%20at%2010.07.32%E2%80%AFAM.png)



---

이후 빌드해서 테스트하면 됩니다.

에러가 발생할 경우 flutter clean 을 수행해보는것도 좋은 방법입니다.