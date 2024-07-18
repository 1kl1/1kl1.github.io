```dart
class AnimatedBuilder extends ListenableBuilder {
  const AnimatedBuilder({
    super.key,
    required ` animation,
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

```dart
abstract class Animation<T> extends Listenable implements ValueListenable<T> {
  const Animation();

  factory Animation.fromValueListenable(ValueListenable<T> listenable, {
    ValueListenableTransformer<T>? transformer,
  }) = _ValueListenableDelegateAnimation<T>;
	// override from ValueListenable 
  @override
  void addListener(VoidCallback listener);

  // override from ValueListenable 
  @override
  void removeListener(VoidCallback listener);

  void addStatusListener(AnimationStatusListener listener);

  void removeStatusListener(AnimationStatusListener listener);

  AnimationStatus get status;
	
  // override from ValueListenable 
  @override
  T get value;

  bool get isDismissed => status == AnimationStatus.dismissed;

  bool get isCompleted => status == AnimationStatus.completed;

  @optionalTypeArgs
  Animation<U> drive<U>(Animatable<U> child) {
    assert(this is Animation<double>);
    return child.animate(this as Animation<double>);
  }

  @override
  String toString() {
    return '${describeIdentity(this)}(${toStringDetails()})';
  }

  
  String toStringDetails() {
    return switch (status) {
      AnimationStatus.forward   => '\u25B6', // >
      AnimationStatus.reverse   => '\u25C0', // <
      AnimationStatus.completed => '\u23ED', // >>|
      AnimationStatus.dismissed => '\u23EE', // |<<
    };
  }
}
```

```dart
abstract class Listenable {
  const Listenable();

  factory Listenable.merge(Iterable<Listenable?> listenables) = _MergingListenable;

  void addListener(VoidCallback listener);

  void removeListener(VoidCallback listener);
}
```



```dart
class AnimationController extends Animation<double>
    with
        AnimationEagerListenerMixin,
        AnimationLocalListenersMixin,
        AnimationLocalStatusListenersMixin {
  AnimationController({
    double? value,
    this.duration,
    this.reverseDuration,
    this.debugLabel,
    this.lowerBound = 0.0,
    this.upperBound = 1.0,
    this.animationBehavior = AnimationBehavior.normal,
    required TickerProvider vsync,
  })  : assert(upperBound >= lowerBound),
        _direction = _AnimationDirection.forward {
    if (kFlutterMemoryAllocationsEnabled) {
      _maybeDispatchObjectCreation();
    }
    _ticker = vsync.createTicker(_tick);
    _internalSetValue(value ?? lowerBound);
  }

  AnimationController.unbounded({
    double value = 0.0,
    this.duration,
    this.reverseDuration,
    this.debugLabel,
    required TickerProvider vsync,
    this.animationBehavior = AnimationBehavior.preserve,
  })  : lowerBound = double.negativeInfinity,
        upperBound = double.infinity,
        _direction = _AnimationDirection.forward {
    if (kFlutterMemoryAllocationsEnabled) {
      _maybeDispatchObjectCreation();
    }
    _ticker = vsync.createTicker(_tick);
    _internalSetValue(value);
  }

  void _maybeDispatchObjectCreation() {
    if (kFlutterMemoryAllocationsEnabled) {
      FlutterMemoryAllocations.instance.dispatchObjectCreated(
        library: _flutterAnimationLibrary,
        className: '$AnimationController',
        object: this,
      );
    }
  }

  final double lowerBound;

  final double upperBound;

  final String? debugLabel;

  final AnimationBehavior animationBehavior;

  Animation<double> get view => this;

  Duration? duration;

  Duration? reverseDuration;

  Ticker? _ticker;

  void resync(TickerProvider vsync) {
    final Ticker oldTicker = _ticker!;
    _ticker = vsync.createTicker(_tick);
    _ticker!.absorbTicker(oldTicker);
  }

  Simulation? _simulation;

  @override
  double get value => _value;
  late double _value;

  set value(double newValue) {
    stop();
    _internalSetValue(newValue);
    notifyListeners();
    _checkStatusChanged();
  }

  void reset() {
    value = lowerBound;
  }

  double get velocity {
    if (!isAnimating) {
      return 0.0;
    }
    return _simulation!.dx(lastElapsedDuration!.inMicroseconds.toDouble() /
        Duration.microsecondsPerSecond);
  }

  void _internalSetValue(double newValue) {
    _value = clampDouble(newValue, lowerBound, upperBound);
    if (_value == lowerBound) {
      _status = AnimationStatus.dismissed;
    } else if (_value == upperBound) {
      _status = AnimationStatus.completed;
    } else {
      _status = (_direction == _AnimationDirection.forward)
          ? AnimationStatus.forward
          : AnimationStatus.reverse;
    }
  }

```

