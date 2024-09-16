---
title: Flutter 개발환경 설정
date: 2024-09-15 00:00:00
categories: [개발, flutter]
tags: [개발, flutter, 튜토리얼, 개발환경 설정]
description: Flutter 개발환경 설정하는법을 설명합니다. windows와 osx 두 케이스 모두에서 설명합니다.
---

# Flutter 개발환경 설정



## 사전 준비사항

권장되는 PC사양은 다음과 같습니다.

| Requirement                  |      Minimum      |    Recommended    |
| :--------------------------- | :---------------: | :---------------: |
| CPU Cores                    |         4         |         8         |
| Memory in GB                 |         8         |        16         |
| Display resolution in pixels | WXGA (1366 x 768) | FHD (1920 x 1080) |
| Free disk space in GB        |       44.0        |       70.0        |





## Mac os

https://docs.flutter.dev/get-started/install/macos/mobile-android

해당 글을 보고 하나하나 따라하면 됩니다.

정리하자면,

1. Rosetta2 설치
2. Android Studio 설치
3. git 설치
4. vscode 설치
5. Flutter sdk 다운로드
6. PATH에 sdk 추가
7. Android Studio에 해당 컴포넌트가 설치되었는지 확인
   1. **Android SDK Platform, API 35.0.1**
   2. **Android SDK Command-line Tools**
   3. **Android SDK Build-Tools**
   4. **Android SDK Platform-Tools**
   5. **Android Emulator**
8. 작동하는 에뮬레이터 셋팅
9. Android license 동의
10. Flutter doctor 실행해서 설치 검증

```shell
Running flutter doctor...
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.24.3, on macOS 14.4.0 23E214 darwin-arm64, locale en)
[✓] Android toolchain - develop for Android devices (Android SDK version 35.0.1)
[!] Chrome - develop for the web
[!] Xcode - develop for iOS and macOS (Xcode not installed)
[✓] Android Studio (version 2024.1)
[✓] VS Code (version 1.93)
[✓] Connected device (1 available)
[✓] Network resources
```

다음과 같이 나오면 Flutter로 android 개발 준비 완 입니다.



## Windows

1. Android Studio 설치
2. git 설치
3. vscode 설치
4. Flutter sdk 다운로드
5. PATH에 sdk 추가
6. Android Studio에 해당 컴포넌트가 설치되었는지 확인
   1. **Android SDK Platform, API 35.0.1**
   2. **Android SDK Command-line Tools**
   3. **Android SDK Build-Tools**
   4. **Android SDK Platform-Tools**
   5. **Android Emulator**
7. 작동하는 에뮬레이터 셋팅
8. Android license 동의
9. Flutter doctor 실행해서 설치 검증

```shell
Running flutter doctor...
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.24.3, on Microsoft Windows 11 [Version 10.0.22621.3155], locale en)
[✓] Windows version (Installed version of Windows is version 10 or higher)
[✓] Android toolchain - develop for Android devices (Android SDK version 35.0.1)
[!] Chrome - develop for the web
[!] Visual Studio - develop Windows apps
[✓] Android Studio (version 2024.1)
[✓] VS Code (version 1.93)
[✓] Connected device (1 available)
[✓] Network resources


! Doctor found issues in 2 categories.
```

다음과 같이 나오면 Flutter로 android 개발 준비 완 입니다.