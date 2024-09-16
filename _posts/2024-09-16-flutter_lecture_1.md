---
title: Dart 언어 기초. Dart의 Future, await, async
date: 2024-09-15 00:01:00
categories: [개발, flutter]
tags: [개발, flutter, 튜토리얼, Dart]
description: Flutter 개발 언어인 Dart 언어의 기본 문법에 대해 설명합니다. Dart만 가지고있는 특징적인 문법과 Future에 대해서 설명합니다. Nullable, late, final, const, if, else, extend, mixin, abstract, async, await, stream, future
---

[TOC]

# Dart란?

**Dart 언어는 클라이언트 개발에 최적화된 언어입니다.** 

Dart는 빠른 개발(1초 미만의 상태 유지 핫 리로드)과 다양한 컴파일 대상(웹, 모바일, 데스크톱)에서 고품질의 결과물을 제공하는 것을 목표로 합니다.

Java처럼 AOT와 JIT컴파일러가 동시에 존재해서 다양한 클라이언트에서 실행시킬 수 있는게 특징입니다.



**정적 타입 검사를 사용**합니다

C, C++, JAVA처럼 변수에 들어갈 값에 타입을 미리 정의합니다.

하지만 `dynamic` 타입을 사용해서 런타임 타입 검사를 사용할 수도 있고, 굉장히 강력한 **타입추론**기능을 가지고 있습니다.



**Null Safety**

개발자가 명시하지 않는 한 변수는 null값을 가질 수 없습니다. 

이 기능을 통해 런타임에서 발생할 수 있는 null 관련 오류를 방지할 수 있습니다.





## Dart 시작해보기

https://dartpad.dev/

```dart
void main() {
  print('Hello, World!');
}
```



# 변수

- String

- int

- bool

- double

- List (Ordered)

- Set (UnOrdered)

- Map

- Object

- dynamic

  

```dart
var name = 'Voyager I';
var year = 1977;
var isDone = true;
var antennaDiameter = 3.7;
var flybyObjects = ['Jupiter', 'Saturn', 'Uranus', 'Neptune'];
var image = {
  'tags': ['saturn'],
  'url': '//path/to/saturn.jpg'
};
```

var = 타입 추론으로 결정. 결정 안되면 dynamic



```dart
String name = 'Voyager I';
int year = 1977;
bool isDone = true;
double antennaDiameter = 3.7;
List<String> flybyObjects = ['Jupiter', 'Saturn', 'Uranus', 'Neptune'];
Set<String> imSet = {"Jupiter", "Saturn"};
Map<String, dynamic> image = {
  'tags': ['saturn'],
  'url': '//path/to/saturn.jpg'
};
```



## Nullable?

{TYPE} + ? = Nullable

```dart
String? name = null;
String name = null; // Error
ServerResponse? response = res.ok ? ServerResponse.fromRes(res) : null;
```



## Late

late + {TYPE} = lazy initialization

- Declaring a non-nullable variable that's initialized after its declaration.
- Lazily initializing a variable.

```dart
late String description;

void main() {
  description = 'Feijoada!';
  print(description);
}
```



> initialize 하지 않고 사용할 시 Runtime Error 발생!!



**추천 사항**

예측하기 힘든 late를 쓰기보다 nullable하게 처리해서 null일경우 예외처리

```dart
String? description;

void main(){
  description = "Hello!";
  print(description ?? "I'm null!!");
}
```



그럼에도 사용하는게 유용할 때가 있는데,

변수를 사용할 수도 있고 사용하지 않을수도 있는데, 초기화하는게 비용이 클 경우에 유용하게 쓸 수 있습니다.

```dart
// This is the program's only call to readThermometer().
late String temperature = readThermometer(); // Lazily initialized.
```

이 경우 temperature를 사용하지 않으면 `readThermometer`가 실행되지 않기 때문입니다.



## Final, Const



```dart
final name = 'Bob'; // Without a type annotation
final String nickname = 'Bobby';


name = "New Bob" // Error: a final variable can only be set once.
```

- instance 변수를 final으로 정의함으로서 instance를 immutable하게 유지할 수 있습니다.



```dart

const bar = 1000000; // Unit of pressure (dynes/cm2)
const double atm = 1.01325 * bar; // Standard atmosphere

```

- const가 compile-time에 결정되기 때문에 const끼리 연산을 const로 할 수 있습니다.

그래서 이런 연산도 가능합니다.

```dart

const Object i = 3; // Where i is a const Object with an int value...
const aList = [i as int]; // Use a typecast.
const aMap = {if (i is int) i: 'int'}; // Use is and collection if.
const aSet = {if (list is List<int>) ...list}; // ...and a spread.

```



## 변수 실습

1. `List<dynamic>` 타입의 변수를 선언하고 다음 조건에 맞추어 값을 초기화 해 주세요.
   - int, bool, String, double 타입의 값을 2개씩 가져야 합니다.

2. (1)의 변수에서 int, double 타입의 변수만 추출하여 새로운 리스트를 선언해주세요.
   - List의 where 내장 함수를 사용합니다.
   - where함수의 리턴값이 iterable이므로 iterable을 toList() 함수로 리스트로 만들어주세요.

> `where(bool Function(T element) test)`은 test가 true인 element만 필터링해줍니다.
>
> ex)
>
> ```dart
> final a = [1, 2, 3, 4];
> final b = a.where((e) => e > 2); // b = [3, 4];
> ```
>
> `is` 키워드로 타입을 확인할 수 있습니다.
>
> ```dart
> final a = 1;
> print(a is int); // true
> ```

3. (2)의 변수들의 총합을 구해주세요,
   - List의 fold 내장 함수를 사용합니다.

> `fold(T initialValue, T Function(T,element) combine)`
>
> fold의 두번째 인자인 Function combine은 두가지 인자를 받는데, 첫번째 인자는 그 전 element의 결과값이고 두번째 인자는 element로 주어집니다. 
>
> ex)
>
> ```dart
> final a = [1,2,3,4];
> final avg = a.fold(0,(previous, element) => previous + element) / a.length;
> ```

 

# 연산자 

- +, -, /, *
- +=. -=, /=, *=
- %, %=
- ~, ~=
- ??
- ... (spread operator)

```dart
final share = 3 ~ 2;
final remainder = 3 % 2;
final imNotNull = imNullable ?? imSubstitute;
final oldList = [3, 4];
final newList = [1, 2, ...oldList]
```



# Control Flow

- if
- else
- for
- while
- break, continue
- switch case

```dart
if (year >= 2001) {
  print('21st century');
} else if (year >= 1901) {
  print('20th century');
}

for (final object in flybyObjects) {
  print(object);
}

for (int month = 1; month <= 12; month++) {
  print(month);
}

while (true) {
  year += 1;
  if(year < 2016) break;
}

```

* **foreach** 를 직관적으로 지원합니다



## Collection if, collection for



```dart

var nav = ['Home', 'Furniture', 'Plants', if (promoActive) 'Outlet'];
var nav = ['Home', 'Furniture', 'Plants', if (login case 'Manager') 'Inventory'];
var listOfInts = [1, 2, 3];
var listOfStrings = ['#0', for (var i in listOfInts) '#$i'];

```

- collection 내부에서 if, for이 사용 가능합니다.
- 상황에 맞는 UI 빌드에 용이합니다.



## Control Flow 실습

1.  collection 내부에서 for를 사용해서 1부터 10까지의 `int`를 가지는 `List`를 선언 해 주세요.
2. (1)의 리스트를 순회하는 for문을 구현한 후, 짝수만 터미널에 출력 해 주세요.
3. (2)의 순회문에서 element가 9보다 크거나 같을 시 순회를 중단 해 주세요.





# Function

{TYPE} 함수명(함수인자){} 로 정의합니다.

```dart
int fibonacci(int n) {
  if (n == 0 || n == 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

var result = fibonacci(20);

var funcResult = (){
  ...Some Operation
}();

flybyObjects.where((name) => name.contains('turn')).forEach(print);
```

* 함수 이름을 안붙이는 anonymous 함수도 지원합니다.
* Arrow syntax가 존재합니다.
  * 함수가 한줄일 경우에 용이합니다.
  * Mapping, filtering하는 함수에서 사용하기에 적합합니다.



# Import

import ... 로 다른 라이브러리의 API를 사용할 수 있습니다.

```dart
import 'dart:developer';
  
  
import 'dart:math' as math;

math.pi // 3.1415...
math.log // log


import 'package:test/test.dart' show Test;

import 'path/to/my_other_file.dart' hide SomeThing;


import 'large_package.dart' deferred as hello;

Future<void> greet() async {
  await hello.loadLibrary();
  hello.printGreeting();
}


```

- `show`와 `hide`키워드를 통해서 특정 부분만 숨기거나 import가 가능합니다.
- `as` 키워드를 통해서 특정 라이브러리를 키워드로 import하는게 가능합니다.
- `deferred as` 키워드를 통해서 lazy 하게 import가 가능합니다.
  - `loadLibrary`를 통해서 라이브러리를 로드할 수 있습니다.



# 객체

- 속성
- 함수
- 생성자

```dart
class Spacecraft {
  String name;
  DateTime? launchDate;

  int? get launchYear => launchDate?.year;

  Spacecraft(this.name, this.launchDate) {}


  void describe() {
    print('Spacecraft: $name');
    var launchDate = this.launchDate;
    if (launchDate != null) {
      int years = DateTime.now().difference(launchDate).inDays ~/ 365;
      print('Launched: $launchYear ($years years ago)');
    } else {
      print('Unlaunched');
    }
  }
}
```

- Java랑은 다르게 public, protected, private이 없습니다.
- _(underscore)로 시작하는 변수가 private입니다.



## Enum

열거형은 다음과 같이 사용할 수 있습니다.

```dart

enum Languages {
  en("English"),
  ko("한국어"),
  jp("日本語");

  final String label;
  
  bool get isKor => label == "한국어";

  const Languages(this.label);
}

enum SimpleEnum{
  type1, type2, type3;
}

```



## 상속

### Extends

```dart
class Orbiter extends Spacecraft {
  double altitude;

  Orbiter(super.name, DateTime super.launchDate, this.altitude);
}
```

- dart는 하나의 객체를 상속할 수 있습니다.



### Mixin 

```dart
mixin Piloted {
  int astronauts = 1;

  void describeCrew() {
    print('Number of astronauts: $astronauts');
  }
}


class PilotedCraft extends Spacecraft with Piloted {
  // ···
}
```

- mixin은 여러 클래스를 상속할 수 있게 해 줍니다.

- type은 extends를 따라가고(Spacecraft), 기능은 with 뒤에 뒤따라오는 클래스들의 기능을 사용할 수 있게 해 줍니다.

  

### Abstract

```dart
abstract class Describable {
  void describe();

  void describeWithEmphasis() {
    print('=========');
    describe();
    print('=========');
  }
}
```

- 추상 클래스는 구현하지 않은 함수를 가질 수 있고
- 상속한 후 구현해주면 됩니다.





## 클래스 실습

1. `int` id, `String` name, `double` value, 총 3가지 필드를 가지는 클래스 하나를 선언 해 주세요.
2. value 값을 N배해 주는 함수 `void multiple(int N)`을 작성해주세요.
3. 해당 클래스를 상속하는 다른 클래스를 선언해 보세요.





# Futures

Dart는 기본적으로 Synchronous 하게 실행됩니다.

API요청이나 일부 Asynchronous 한 작업이 필요할 땐 Async, Await을 이용해서 callback 지옥으로부터 벗어날 수 있습니다.



## Async, Await

```dart
const oneSecond = Duration(seconds: 1);
// ···
Future<void> printWithDelay(String message) async {
  await Future.delayed(oneSecond);
  print(message);
}

Future<void> printWithDelay(String message) {
  return Future.delayed(oneSecond).then((_) {
    print(message);
  });
}

// 위의 두 코드는 같은 코드입니다.
```

함수를 선언할 때 `async` 키워드를 추가 해 주면 

함수 내에서 `await` 키워드를 사용할 수 있고,

`await`이 끝나야 그 다음줄이 실행됩니다.



##  Stream

```dart
Stream<String> report(Spacecraft craft, Iterable<String> objects) async* {
  for (final object in objects) {
    await Future.delayed(oneSecond);
    yield '${craft.name} flies by $object';
  }
}

await for (String res in report(craft, [])) {
  // Executes each time the stream emits a value.
}
```

`async*` 키워드를 통해서 `Stream`을 구현할 수 있습니다.



## Future 실습

1. `int`인자 N을 하나 받아서 N초 뒤에 해당 값 N을 return 하는 Future 함수를 하나 선언해 주세요.

2. 아래의 별을 터미널에 출력하되, (1)에서 만든 함수를 이용해서 한줄의 별 출력이 끝나면 해당 줄에 출력한 별의 갯수의 합(K) 만큼 delay를 해 주세요.

   ```shell
   *
   **
   ***
   ****
   *****
   ```

> ```dart
> Future.delayed(Duration(seconds: 1)); // 해당 Future는 1초의 delay를 가집니다. 
> ```



