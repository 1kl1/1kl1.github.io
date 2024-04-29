---
title: Flutter overlay 사용법
date: 2023-12-15 00:00:00
categories: [개발, flutter]
tags: [개발, flutter, overlay, 팁, 노하우]
description: flutter에서 만들 수 있는 다양한 overlay에 대해 담은 글입니다.CompositedTransformTarget, CompositedTransformFollower, OverlayPortal, OverlayEntry사용법도 다루고 있습니다.


---

flutter로 개발 하다 보면 특정 상황에서 다른 widget보다 위에 위젯이 있어야 하거나,

전체 화면중에서 가장 위에 위젯이 있어야 하는 상황이 종종 발생합니다.

예를 들면 popup, tooltip, 알림 등이 있습니다.

그 때 사용하는것이 overlay입니다.

ovelay는 buildcontext에서 삽입되는것이 아니라, overlaystate에서 삽입됩니다.

`Overlay.of(context)`로 접근 가능합니다.

 **Overlay**,  **OverlayEntry** 이 두가지를 이용하여 간단한 overlay를 띄우는 방법과,

**CompositedTransformTarget**, **CompositedTransformFollower**, **OverlayPortal**

 이 세가지를 이용하여 다소 복잡한 overlay를 띄우는 방법에 대해서 알아보겠습니다.



## Overlay, OverlayEntry

> 예시는 `go_router: ^12.1.1` 를 사용하여 구현하였습니다.

구현하고자 하는 효과는 다음과 같습니다.

- 특정 시간이 지나면 overlay가 사라져야 함.
- overlay에는 단순한 string을 입력하여 보여줄 수 있어야 함.
- 동시에 여러개의 overlay를 보여주더라도 오류가 없어야 함.
- overlay가 닫히기까지를 future로 받아볼 수 있어야 함.

해당 효과를 구현하기 위해서 overlayEntry를 overlayState에 insert하고,

특정 시간이 지나면 해당 overlayEntry를 remove하게 합니다.

future로 overlay가 닫히기까지를 받아보기 위해서 Completer로 동시에 기용하였습니다.

```dart
class SimpleNotify {
  Future<void> show(String text) async {
    final completer = Completer();
    OverlayEntry? entry;
    entry = OverlayEntry(
      builder: (_) {
        return _SimpleNotifyContainer(
          text: text,
          duration: const Duration(seconds: 2),
          onRemoved: () {
            entry?.remove();
            completer.complete();
          },
        );
      },
    );
		final goRouter = [Dependency Injection으로 주입해 놓은 instance를 받아옵니다.]
		goRouter.overlayState?.insert(entry);
    return completer.future;
  }
}

```

차례차례 순서를 짚어보면,

1. OverlayEntry를 만들어 줍니다. builder 메서드는 widget을 return합니다.
2. OverlayState를 받아와서 만들어 준 entry를 insert 해 줍니다.
3. completer를 widget에 넘겨 주고, widget에서는 remove 될 때 completer를 complete해 주고, show에서는 completer의 future를 return해 줍니다.



`_SimpleNotifyContainer` 는 statefulwidget으로, 다음과 같이 구현했습니다.

```dart
class _SimpleNotifyContainer extends StatefulWidget {
  const _SimpleNotifyContainer({
    required this.text,
    required this.duration,
    required this.onRemoved,
  });

  final String text;
  final Duration duration;
  final VoidCallback? onRemoved;

  @override
  _SimpleNotifyState createState() => _SimpleNotifyState();
}

class _SimpleNotifyState extends State<_SimpleNotifyContainer> {
  final positionKey = GlobalKey();
  bool isVisible = false;

  @override
  void initState() {
    Future.delayed(const Duration(milliseconds: 1), () {
      setState(() => isVisible = true);
    });
    Future.delayed(widget.duration, () {
      setState(() => isVisible = false);
      Future.delayed(const Duration(milliseconds: 300), () {
        widget.onRemoved?.call();
      });
    });

    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    var width = 34 + widget.text.length * 14.0;

    if (width > context.getWidth - 34) {
      width = context.getWidth - 34;
    }
    return Center(
      key: positionKey,
      child: AnimatedOpacity(
        opacity: isVisible ? 1 : 0,
        curve: Curves.easeInOutQuart,
        duration: const Duration(milliseconds: 300),
        child: Material(
          color: Colors.transparent,
          borderRadius: BorderRadius.circular(5),
          child: Wrap(
            children: [
              Container(
                margin: const EdgeInsets.symmetric(horizontal: 17),
                width: width,
                padding: const EdgeInsets.symmetric(
                  horizontal: 17,
                  vertical: 8,
                ),
                decoration: BoxDecoration(
                  color: Colors.black,
                  borderRadius: BorderRadius.circular(5),
                  boxShadow: [
                    BoxShadow(
                      color: Colors.white.withOpacity(0.3),
                      blurRadius: 10,
                      offset: const Offset(0, 1),
                    ),
                  ],
                ),
                child: Center(
                  child: Text(
                    widget.text,
                    style: const TextStyle(color: Colors.white),
                    textAlign: TextAlign.center,
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

```



핵심적인 부분은 state와 initState입니다.

```dart
  final positionKey = GlobalKey();
  bool isVisible = false; 

  @override
  void initState() {
    Future.delayed(const Duration(milliseconds: 1), () {
      setState(() => isVisible = true);
    });
    Future.delayed(widget.duration, () {
      setState(() => isVisible = false);
      Future.delayed(const Duration(milliseconds: 300), () {
        widget.onRemoved?.call();
      });
    });

    super.initState();
  }
```

1. fadeIn되는 효과를 위해서 isVisible을 처음에는 false, 직후 isVisible을 true로 해 주었습니다.
2. 지정한 duration 이후에 isVisible을 false로 해 주고
3. 300ms뒤에 onRemoved 메서드를 호출 해 주 었습니다.(300ms는 fadeout까지의 시간입니다.)

이 방법으로 원하는 요구사항을 만족하는 overlay를 구현할 수 있었습니다.



## **CompositedTransformTarget**, **CompositedTransformFollower**, **OverlayPortal**

가장 먼저 각각의 역할에 대해 살펴보겠습니다.

