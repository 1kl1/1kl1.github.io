---
title: flutter 카카오 로그인 연동기
date: 2024-01-06 11:52:00
categories: [개발, flutter]
tags: [개발, flutter, 카카오톡 로그인, 카카오톡 연동, oauth]

---

지금 일하고 있는 스타트업에서 피봇을 하게 되었고 새로 진행하는 프로젝트에 카카오 로그인을 도입하게 되었습니다.

이번 글에서는 카카오 로그인을 도입하는 방법을 한단계 한단계 기록해보려고 합니다.



## 개요

### 카카오 로그인과 카카오 싱크

카카오 oauth를 통해서 로그인하고 카카오 유저의 정보를 읽어오는 방법으로는 두가지가 있습니다.

카카오 로그인 vs 카카오 싱크 입니다.

카카오 로그인과 카카오 싱크의 차이점은 이렇게 명시되어 있습니다.

<img src="../assets/img/2024-01-03-kakao%20login/image-20240103133905544.png" alt="image-20240103133905544" style="zoom:50%;" />

요약하자면

기존 카카오 로그인 + 추가적인 약관동의(약관 동의 시 닉네임, 프로필 이미지 등 유저의 정보를 선택이 아닌 필수로 가져올 수 있습니다.)

= 카카오 싱크 라고 생각하면 되겠습니다.

### 토큰에 대한 정보

엑세스 토큰, 리프레시 토큰, ID 토큰 세가지가 있습니다. 

Access Token을 통해서 카카오에서 제공하는 유저 정보를 가져올 수 있으며

Access Token은 만료 기간이 짧기때문에 Refresh Token을 통해서 Access Token을 갱신 할 수 있습니다.

<img src="../assets/img/2024-01-03-kakao%20login/image-20240103134116835.png" alt="image-20240103134116835" style="zoom:50%;" />

### 로그인 과정

아래 과정은 일반적인 Web이나 Flutter Web app에서의 Oauth 처리 방식입니다. Redirect URI를 사전에 카카오 플랫폼에 등록시켜주어야 합니다.

1. 로그인 버튼 클릭
2. 인증 및 동의하는 창
3. 로그인 및 동의
4. 사전에 등록한 redirect URI로 redirect
5. redirect parsing
6. 토큰 발급 요청
7. 토큰 발급

 android, ios의 경우는 카카오에서 제공하는 Flutter package를 이용하면 redirect가 아니라 custom url(딥링크)를 통해서 해당 로그인이 진행됩니다.

1. 로그인 버튼 클릭
2. 인증 및 동의 창
3. 로그인 및 동의
4. custom url로 redirect
5. kakao package에서 parsing
6. 토큰 발급 요청
7. 토큰 발급
8. 토큰 저장(자동으로 이루어짐)

토큰 저장도 자동으로 처리해주므로 다른 저장소 패키지를 이용하지 않아도 됩니다.

### 회원가입

회원가입은 로그인 후 자체적으로 구현하게 됩니다.

1. 로그인 후 토큰으로 사용자 정보 가져오기
2. 사용자 정보를 통해 회원가입 처리

추후 이용약관 등을 이 과정에서 직접 구현하면 되겠습니다.



## 직접 해보기

일단 패키지를 pubspec.yaml에 추가 해 줍니다.

해당 패키지를 추가해주면 다른 카카오 공유/로그인 등을 알아서 dependency 처리를 해 줍니다.

```yaml
kakao_flutter_sdk: ^1.8.0
```

해당 패키지는 아래의 패키지들에 dependency를 가지고 있습니다.

```yaml
dio
json_annotation
platform
shared_preference
crypto
encrypt
```



### 플랫폼에 등록하기

kakao developers에 로그인/회원가입 후 console에 들어가서 어플리케이션을 추가 해 줍니다.

https://developers.kakao.com/console/app

어플리케이션을 추가 후 플랫폼을 등록 해 주어야 합니다.



<img src="../assets/img/2024-01-03-kakao%20login/image-20240103165001670.png" alt="image-20240103165001670" style="zoom:50%;" />

패키지명은 android>app>build.gradle에서 com.xxx.xxx를 찾아서 확인하면 됩니다.

이미지상의 키 해시는 제 키가 아니고 카카오에서 hint로 적어 둔 것이니 유효하지 않습니다.

직접 본인의 키 해시를 받아서 등록 해 주어야 합니다.



### 키 해시 받아오기(osx)

우선 openssl 이 설치 되어 있어야 합니다.

debugging용 key를 받아오는 명령어 입니다. 일반적으로 android studio를 설치 할 경우 아래의 명령어가 유효합니다.

``` shell
keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore -storepass android -keypass android | openssl sha1 -binary | openssl base64
```



release용 key는 다소 복잡합니다.

일단 keystore를 생성해야함. keystore가 있다면 하단의 keystore 생성 과정을 스킵 하셔도 됩니다.

flutter 프로젝트를 android studio로 열고

이후 상단의 build > generate signed bundle/apk 클릭.

create new해서 keystore를 생성하면 됩니다.

<img src="../assets/img/2024-01-03-kakao%20login/image-20240103185453669.png" alt="image-20240103185453669" style="zoom:40%;" />

이후에 아래의 명령어에서 RELEASE_KEY_ALIAS, RELEASE_KEY_PATH를 상기에서 생성했던 값을 넣어서 진행하면 hash key를 얻을 수 있습니다.

```shell
keytool -exportcert -alias <RELEASE_KEY_ALIAS> -keystore <RELEASE_KEY_PATH> | openssl sha1 -binary | openssl base64
```

플랫폼 등록 후에는 카카오 로그인을 활성화 해 주면 됩니다.

![image-20240103190301755](../assets/img/2024-01-03-kakao%20login/image-20240103190301755.png)

좌측의 탭에서 동의항목으로 들어가 동의 항목을 입맛에 맞게 수정할 수 있습니다.

<img src="../assets/img/2024-01-03-kakao%20login/image-20240103190343038.png" alt="image-20240103190343038" style="zoom:50%;" />

### 커스텀 URL Scheme

네이티브 앱 서비스에서 카카오 로그인 API를 사용하려면 [커스텀 URL 스킴(Custom URL Scheme)](https://developers.kakao.com/docs/latest/ko/flutter/getting-started#setup-scheme)을 설정해야 합니다. 아래의 디바이스 환경별 프로젝트 설정 방법을 참고합니다.

- Android: [커스텀 URL 스킴](https://developers.kakao.com/docs/latest/ko/flutter/getting-started#project-scheme)
- iOS: [커스텀 URL 스킴](https://developers.kakao.com/docs/latest/ko/flutter/getting-started#project-scheme-ios)

커스텀 URL 스킴(Custom URL Scheme)은 사용자가 Android와 iOS 환경에서 카카오톡으로 로그인 후 서비스 앱으로 돌아오거나, 카카오톡 메시지 버튼 또는 링크로 서비스의 앱을 실행하는 데 사용됩니다.

이 글에서는 android를 설정하는 방법만 다루겠습니다.

```xml
<application>
	...
    <!-- 카카오 로그인 커스텀 URL 스킴 설정 -->
    <activity 
        android:name="com.kakao.sdk.flutter.AuthCodeCustomTabsActivity"
        android:exported="true">
        <intent-filter android:label="flutter_web_auth">
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />

            <!-- "kakao${YOUR_NATIVE_APP_KEY}://oauth" 형식의 앱 실행 스킴 설정 -->
            <!-- 카카오 로그인 Redirect URI -->
            <data android:scheme="kakao${YOUR_NATIVE_APP_KEY}" android:host="oauth"/>
        </intent-filter>
    </activity>
  ...
</application>
```

위와 같이 정의 해 주면 됩니다.

제 프로젝트에선 github에 업로드 하고 있기 때문에 해당 key부분을 숨겨야 해서 manifestPlaceholder를 이용했습니다.

해당 방법을 적용하기위해서 data를 다음과 같이 수정했습니다.

```xml
<data android:scheme="kakao${kakaoApiKey}" android:host="oauth"/>
```

이후 build.gradle에서 해당 코드들을 추가 했습니다.

주의해야 할 점은 manifestPlaceholder에 =이 아니라 +=을 해 주어야 합니다.

```text

...
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')

if (keystorePropertiesFile.exists()) {
   keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}
def kakaoApiKey = keystoreProperties.getProperty('kakao.api.key')

android{
	...
    defaultConfig {
				...
        manifestPlaceholders += [kakaoApiKey:kakaoApiKey]
				...
    }
	...
}
```

이후 android level에 key.properties 파일을 추가 해 주었습니다.

kakao.api.key 뿐만 아니라 release용 key까지 추가하여 구성 했습니다.

<img src="../assets/img/2024-01-03-kakao%20login/image-20240104104704820.png" alt="image-20240104104704820" style="zoom:50%;" />

### Flutter 코드로 구현하기

로그인은 다음의 flow로 진행됩니다.

<img src="../assets/img/2024-01-03-kakao%20login/image-20240104095139182.png" alt="image-20240104095139182" style="zoom:50%;" />

```dart
try {
  OAuthToken token = await UserApi.instance.loginWithKakaoTalk();
  print('카카오톡으로 로그인 성공 ${token.accessToken}');
} catch (error) {
  print('카카오톡으로 로그인 실패 $error');
}
```

 발급 발급된 토큰은 `TokenManagerProvider`에 지정된 토큰 저장소에 자동으로 저장됩니다.

로그인 되어있는 정보가 있다면 다음의 명령어로 가져올 수 있습니다.

```dart
final token = await TokenManagerProvider.instance.manager.getToken();
```

로그아웃 처리를 위해서 다음과 같이 token 정보를 지워줄 수도 있습니다. 

```dart
TokenManagerProvider.instance.manager.clear();
```



마지막으로, 좀 더 사용자에게 매끄러운 경험을 제공 해주기 위해 카톡이 설치되어있으면 카톡으로 로그인, 그게 아니라면 브라우저를 통해 로그인 할 수 있게 구현합니다.

```dart
await isKakaoTalkInstalled() // kakaotalk이 설치되어 있는지 확인
```

최종적으로 다음과 같이 구현했습니다.

```dart
  void loginWithKakao() async {
    if (await isKakaoTalkInstalled()) {
      try {
        OAuthToken token = await UserApi.instance.loginWithKakaoTalk();
        isLoggedIn = true;
        kakaoToken = token;
        debugPrint('카카오톡으로 로그인 성공 ${token.accessToken}');
      } catch (error) {
        debugPrint('카카오톡으로 로그인 실패 $error');
      }
    } else {
      try {
        OAuthToken token = await UserApi.instance.loginWithKakaoAccount();
        isLoggedIn = true;
        kakaoToken = token;
        debugPrint('카카오계정으로 로그인 성공');
      } catch (error) {
        debugPrint('카카오계정으로 로그인 실패 $error');
      }
    }
  }
```



### UI에 버튼 넣기

 카카오 로그인 버튼을 만드는 UI 규칙도 있습니다. 첫번째 이미지의 button asset을 이용하거나, 아래의 색상 및 가이드라인을 지켜 직접 버튼 UI를 구성하는 방식입니다.

<img src="../assets/img/2024-01-03-kakao%20login/image-20240104100715282.png" alt="image-20240104100715282" style="zoom:50%;" />

<img src="../assets/img/2024-01-03-kakao%20login/image-20240104100739375.png" alt="image-20240104100739375" style="zoom:50%;" />

이 URL에서 더 자세한 내용을 알 수 있습니다. https://developers.kakao.com/docs/latest/ko/kakaologin/design-guide



### 문제 해결

- LateInitializationError

```she
LateInitializationError: Field 'platforms' has not been initialized.
```

해당 에러가 발생할 경우 init을 runApp전에 해주지 않아서 생긴 문제입니다. 

```dart
KakaoSdk.init(nativeAppKey: kakaoAppKey);
```

이 메서드를 runApp전에 실행 시켜주면 해결됩니다.



- error: misconfigured

카카오톡으로 로그인 실패 {error: misconfigured, error_description: invalid android_key_hash or ios_bundle_id or web_site_url}

해당 에러가 생기는 경우는 android_key_hash가 적절하지 않거나 package이름이 틀려서 생기는 경우입니다.

알맞게 설정 해 주고 문제를 해결했습니다.





