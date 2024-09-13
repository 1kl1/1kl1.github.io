---
title: Listenable이란? Animation과 ChangeNotifier를 통해서 알아보자.
date: 2024-09-12 00:00:00
categories: [개발, flutter]
tags: [개발, flutter, Listenable, AnimatedBuilder, AnimationController, ChangeNotifier]
description: Flutter Listenable, Tick, AnimatedBuilder의 원리. flutter decoding 파해치기, AnimationController, SingleTickerProviderStateMixin, AnimatedWidget, ImplicitAnimation, ChangeNotifier




---

## AnimationController와 SingleTickerProviderStateMixin

개발을 하다 보니 1프레임마다 widget을 업데이트 해 주어야 하는 상황이 생겼습니다.

보통 이럴 때에는 1프레임을 정확하게 받기 위해서(사용자 기기의 성능도 다르고 구현 상황에 따라 프레임이 다르기 때문에)

`StatefulWidget`에서 `SingleTickerProviderStateMixin`을 mix 해 주고, mix한 클래스에서 `this`로 tick 정보를 받아서 구현하곤 합니다.

this는 `AnimationController`를 초기화할 때 vsync로 넣어주면 되지요.

정리하면 이렇게 됩니다.

```dart 
class AWidget extends StatefulWidget{
  ...
}

class _AWidgetState extends State<AWidget> with SingleTickerProviderStateMixin{
  late AnimationController _controller;
  
  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
    	duration: const Duration(milliseconds: 1000),
      vsync: this,
    )
  }
}
```

이렇게 초기화 한 `AnimationController`를 이용해서 우리는 `AnimatedBuilder` 안에 animation값을 넣고 프레임마다 animation을 연산 하여 프레임마다 바뀌는 구현을 해 줍니다.

```dart
AnimatedBuilder(
  animation: _animation!,
  builder: (context, child) => CustomPaint(
    painter: LinePainter(_animation!.value),
  ),
)
```



그러면, 실제 `AnimatedBuilder`의 구현은 어떻게 되어있길래, animation 값이 바뀔때마다 내부의 위젯을 업데이트 해 줄 수 있는걸까요?

`StatefulWidget`과 구현이 다르다면, 이렇게 tick마다 업데이트 되는 위젯에 한해서 `setState`를 해 주는 것보다 더 효율적인 방법이 flutter에 존재하는 것 일까요?

## AnimatedBuilder

구현은 다음과 같이 되어있었습니다. (주석 삭제)

```dart
class AnimatedBuilder extends ListenableBuilder {
  const AnimatedBuilder({
    super.key,
    required Listenable animation,
    required super.builder,
    super.child,
  }) : super(listenable: animation);

  Listenable get animation => super.listenable;
  
  @override
  Listenable get listenable => super.listenable;

  @override
  TransitionBuilder get builder => super.builder;
}
```

`AnimatedBuilder`는 `ListenableBuilder` 의 구현에 매우 의존하고 있다는 것을 알았습니다.

`ListenableBuilder`를 알아 볼 차례네요.

```dart
class ListenableBuilder extends AnimatedWidget {
  const ListenableBuilder({
    super.key,
    required super.listenable,
    required this.builder,
    this.child,
  });

  @override
  Listenable get listenable => super.listenable;
  final TransitionBuilder builder;
  final Widget? child;

  @override
  Widget build(BuildContext context) => builder(context, child);
}
```



`AnimatedWidget`에 대해서도 알아봐야겠습니다.  

```dart
abstract class AnimatedWidget extends StatefulWidget {
  const AnimatedWidget({
    super.key,
    required this.listenable,
  });

  final Listenable listenable;

  @protected
  Widget build(BuildContext context);
  @override
  State<AnimatedWidget> createState() => _AnimatedState();

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.add(DiagnosticsProperty<Listenable>('listenable', listenable));
  }
}

```

`AnimatedWidget`은 abstract class ! 였습니다.

StatefulWidget이었고, 사실상 `ListenableBuilder`가 implementation을 하고 있는데, build 메소드가 우리가 인자로 주고있는 builder를 그냥 return하는것 뿐이네요.

우리가 들었던 예시로 보면

```dart
AnimatedBuilder(
  animation: _animation!,
  builder: (context, child) => CustomPaint(
    painter: LinePainter(_animation!.value),
  ),
)
```

여기서의 builder를 그대로 build하는것으로 보입니다.

`StatefulWidget`의 구현에 별 로직이 없었으므로, `AnimatedWidget`의 state에 대해서도 알아봐야겠네요.

```dart
class _AnimatedState extends State<AnimatedWidget> {
  @override
  void initState() {
    super.initState();
    widget.listenable.addListener(_handleChange);
  }

  @override
  void didUpdateWidget(AnimatedWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.listenable != oldWidget.listenable) {
      oldWidget.listenable.removeListener(_handleChange);
      widget.listenable.addListener(_handleChange);
    }
  }

  @override
  void dispose() {
    widget.listenable.removeListener(_handleChange);
    super.dispose();
  }

  void _handleChange() {
    if (!mounted) {
      return;
    }
    setState(() {
      // The listenable's state is our build state, and it changed already.
    });
  }

  @override
  Widget build(BuildContext context) => widget.build(context);
}
```

이렇게 구현이 되어있었습니다.

정리를 하면, `initState`일 때 `listenable`에 `addListener`를 합니다. 

`_handleChange`는 말 그대로.. widget이 mount되어있다면 `setState`만을 해 주고 있네요.

나머지코드는 listenable이 바뀌었다면 `removeListener`와 `addListener`를 해 주고 있는것이고, 위젯이 dispose될 때 `removeListener`를 해 주고 있습니다.

그렇다면 `Listenable`이라는것은 무엇이길래, 값 변화를 감지하고 `addListener`되어있는 함수들은 호출하는 지 궁금해졌습니다.



## Listenable

가장 먼저 listenable을 살펴보겠습니다.

```dart
abstract class Listenable {
  const Listenable();

  factory Listenable.merge(Iterable<Listenable?> listenables) = _MergingListenable;

  void addListener(VoidCallback listener);

  void removeListener(VoidCallback listener);
}
```

이와 같이 아주 간단하게 abstract class인 것을 볼 수 있습니다.

listener를 더하고 뺄 수 있는 함수가 있습니다.

가장 대표적으로` Listenable`를 implements하는 클래스는 `ChangeNotifier`라고 할 수 있습니다.



### ChangeNotifier

`ChangeNotifier`는 change notification을 제공해줄 수 있는 mixin class입니다.

구현을 확인해보면 300줄에 달할정도로 긴 구현임을 확인할 수 있습니다.

핵심적인부분만 글에 담아보겠습니다.

```dart
mixin class ChangeNotifier implements Listenable {
  int _count = 0;

  static final List<VoidCallback?> _emptyListeners = List<VoidCallback?>.filled(0, null);
  List<VoidCallback?> _listeners = _emptyListeners;
  int _notificationCallStackDepth = 0;
  int _reentrantlyRemovedListeners = 0;
  bool _debugDisposed = false;

	...

  @override
  void addListener(VoidCallback listener) {
	...
    if (_count == _listeners.length) {
      if (_count == 0) {
        _listeners = List<VoidCallback?>.filled(1, null);
      } else {
        final List<VoidCallback?> newListeners =
            List<VoidCallback?>.filled(_listeners.length * 2, null);
        for (int i = 0; i < _count; i++) {
          newListeners[i] = _listeners[i];
        }
        _listeners = newListeners;
      }
    }
    _listeners[_count++] = listener;
  }

...

  @override
  void removeListener(VoidCallback listener) {
    for (int i = 0; i < _count; i++) {
      final VoidCallback? listenerAtIndex = _listeners[i];
      if (listenerAtIndex == listener) {
        if (_notificationCallStackDepth > 0) {
          _listeners[i] = null;
          _reentrantlyRemovedListeners++;
        } else {
          _removeAt(i);
        }
        break;
      }
    }
  }


  @protected
  @visibleForTesting
  @pragma('vm:notify-debugger-on-exception')
  void notifyListeners() {
    assert(ChangeNotifier.debugAssertNotDisposed(this));
    if (_count == 0) {
      return;
    }

    _notificationCallStackDepth++;

    final int end = _count;
    for (int i = 0; i < end; i++) {
      try {
        _listeners[i]?.call();
      } catch (exception, stack) {
       ...
      }
    }

    _notificationCallStackDepth--;

		...
  }
}
```



가장 핵심적인 함수는 `addListener`와 `removeListener`, `notifyListeners`입니다. 

abstract였던 함수들이 concrete하게 구현되는 부분들이죠.

`notifyListeners`를 보면 _listener들을 하나한 call해주는 것을 볼 수 있습니다.

실제로 `ChangeNotifier`를 extends해서 쓰는 `ValueNotifier`는 내부의 값이 변할 때 그 값이 예전 값과 같지 않다면 notifyListeners를 호출합니다.  이 함수들을 들여다보면 listener들을 직접 call해주는걸 확인할 수 있었습니다.

listener들은 하나의 리스트로 관리되고 addListener와 removeListener가 그 리스트에 listener들을 추가하고 제거해주는 역할을 해 주는 것이죠.





### Animation

animation은 다소 특이합니다.

처음엔 `ChangeNotifier`와 비슷할것이라고 생각했는데, 아무리 내부 구현을 확인해도 실제 listener를 호출하는 코드는 확인할 수 없었습니다.

정답은,` Animation`은 listener들에게 notify할 때 `AnimationController`의 `AnimationLocalListenerMixin`의 구현에서 notify한다는 것 이었습니다.

```dart
mixin AnimationLocalListenersMixin on Animation<double> {
  List<VoidCallback>? _listeners = [];

  @protected
  void notifyListeners() {
    if (_listeners != null) {
      final List<VoidCallback> localListeners = List<VoidCallback>.from(_listeners!);
      for (final VoidCallback listener in localListeners) {
        try {
          listener();
        } catch (exception, stack) {
          FlutterError.reportError(FlutterErrorDetails(
            exception: exception,
            stack: stack,
            library: 'animation library',
            context: ErrorDescription('while notifying listeners for $runtimeType'),
            informationCollector: () => <DiagnosticsNode>[
              DiagnosticsProperty<Animation<double>>(
                'The $runtimeType notifying listeners was',
                this,
                style: DiagnosticsTreeStyle.errorProperty,
              ),
            ],
          ));
        }
      }
    }
  }

  @override
  void addListener(VoidCallback listener) {
    _listeners!.add(listener);
  }

  @override
  void removeListener(VoidCallback listener) {
    _listeners!.remove(listener);
  }
}
```

실제 구현은 이렇게 되어있었죠. 아까 봤던 `ChangeNotifier`의 구현은 addListener와 removeListener에 좀 더 공을 쓴 느낌이었는데 이건 그냥 간단하게 구현이 되어 있네요.

마찬가지로 listener를 호출하는 방식으로 listen을 구현하고 있습니다. 이번엔 call을 호출하는게 아니라 그냥 함수를 실행하는 방식이네요.

`Listenable`한 건 `Animation`인데 어째서` AnimationController`에서 notify의 실구현이 되어있고 이게 어떻게 작동하는건가요? 라고 묻는다면, `AnimationController`도 `Animation`을 상속하고 있다는 사실을 말씀드리고 싶습니다.

`CurvedAnimation`, `AlwaysStoppedAnimation`... 등도 Animation을 상속하고 있고, `AnimationController`도 `Animation`을 상속하고 있고...  `Animation`은 `Listenable`을 상속하고있고.. 결국 모든게 `Listenable`하지만, 실제로 `Listenable`한게 유효한 대상은 `AnimationController`라고 볼 수 있겠습니다만.. 사실은 `CurvedAnimation`이든 특정 작동하는 `Animation`을 초기화할때 파라미터인 parent 값에 `AnimationControlle`r를 넣어줘야 합니다. 그리고 이 작동하는` Animation`들은 `AnimationWithParentMixin`을 mix하고있고, 이 mixin에서는 작동하는 `Animation`에 추가되는 addListener를 parent에 addListener하도록 hooking해주고 있죠.

결국 구현은 `AnimationController`에 되어 있지만 실제 사용할때에는` Animation`들 모두가 `AnimationControlle`r의 listener구현에 따라 작동한다는 것이고, 그 실제 구현은 `AnimationLocalListenersMixin`에 되어있다는게 결론입니다.



