---
title: Flutter에서 유용하게 사용했던 패키지 UIUX 1편
date: 2023-11-30 00:00:00
categories: [개발, flutter]
tags: [개발, uiux, flutter]
description: flutter shimmer, flutter_staggered_animations, auto_size_text, carousel_slider 사용법, 유용한 팁


---

Flutter의 오픈소스 커뮤니티를 이용해서 이제껏 정말 빠르고 쉽게 개발을 해 왔습니다.

구글에서도 적극적으로 밀고있는 https://pub.dev/ 입니다.

저 또한 잘 사용해 왔었는데, 문득 flutter로 개발해 온 2년간 유용하게 사용해왔던 패키지들을

정리해 놓으면 다른 이들에게 도움이 될 수 있어 좋겠다는 생각이 들어 정리 해 보았습니다.

이외에도 사용하는 패키지가 많지만 소수만 정리 해 보았습니다.

uiux 편 입니다.



## shimmer

![img](../assets/img/2023-11-30-flutter_packages_uiux1/loading_list.gif)

이런 반짝이는 효과를 줄 수 있는 패키지입니다.

```dart
SizedBox(
  width: 200.0,
  height: 100.0,
  child: Shimmer.fromColors(
    baseColor: Colors.red,
    highlightColor: Colors.yellow,
    child: Text(
      'Shimmer',
      textAlign: TextAlign.center,
      style: TextStyle(
        fontSize: 40.0,
        fontWeight:
        FontWeight.bold,
      ),
    ),
  ),
);

```

빈 sized box에 shimmer효과를 줌으로서 로딩을 좀 더 예쁘게 바꿔줄 수 있습니다.



## flutter_staggered_animations

<img src="../assets/img/2023-11-30-flutter_packages_uiux1/card_list.gif" alt="img" style="zoom:33%;" /> <img src="https://github.com/mobiten/flutter_staggered_animations/blob/master/assets/card_grid.gif?raw=true" alt="img" style="zoom:33%;" />

로딩이 끝난 후 item들이 있을 때, item listing을 예쁘게 해줄 수 있는 패키지입니다.

```dart
AnimationLimiter(
  child: ListView.builder(
    itemCount: 100,
    itemBuilder: (BuildContext context, int index) {
      return AnimationConfiguration.staggeredList(
        position: index,
        duration: const Duration(milliseconds: 375),
        child: SlideAnimation(
          verticalOffset: 50.0,
          child: FadeInAnimation(
            child: YourListChild(),
          ),
        ),
      );
    },
  ),
);
```

`AnimationLimiter` 로 listview를 감싸고, item을 `AnimationConfiguration.staggered~` 로 감싼 후 원하는 효과로 감싸주면 효과를 간편하게 구현할 수 있습니다.



## auto_size_text

## <img src="https://raw.githubusercontent.com/leisim/auto_size_text/master/.github/art/maxlines.gif" alt="img" style="zoom:50%;" />

constraint가 있을 때, max line을 넘어가는 글 이면 글씨 크기를 자동으로 줄여주는 패키지입니다.

```dart
AutoSizeText(
  'A really long String',
  style: TextStyle(fontSize: 30),
  minFontSize: 18,
  maxLines: 4,
  overflow: TextOverflow.ellipsis,
)
```

이렇게 간단하게 사용할 수 있습니다.



## carousel_slider

## <img src="https://github.com/serenader2014/flutter_carousel_slider/raw/master/screenshot/indicator.gif" alt="indicator" style="zoom:67%;" />

좌 우로 스크롤 가능한 배너나, pageview등을 쉽게 구현할 수 있게 해 놓은 패키지입니다.

indicator나, center에 있는 item만 enlarge한다던가, autoscroll한다던가 하는 등을 쉽게 구현할 수 있습니다.

```dart
CarouselSlider(
  options: CarouselOptions(height: 400.0),
  items: [1,2,3,4,5].map((i) {
    return Builder(
      builder: (BuildContext context) {
        return Container(
          width: MediaQuery.of(context).size.width,
          margin: EdgeInsets.symmetric(horizontal: 5.0),
          decoration: BoxDecoration(
            color: Colors.amber
          ),
          child: Text('text $i', style: TextStyle(fontSize: 16.0),)
        );
      },
    );
  }).toList(),
)
```

이렇게 구현할 수 있습니다.
