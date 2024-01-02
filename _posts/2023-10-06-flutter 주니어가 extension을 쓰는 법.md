---
title: 2년차 flutter 주니어가 extension를 쓰는 법
date: 2023-10-06 22:30:00
categories: [개발, flutter]
tags: [개발, flutter, dart3, records, extension]

---

dart에는 독특한 문법 하나가 있습니다.`extension`이라는 키워드를 이용해서 기존의 클래스를 `확장` 하는 문법입니다. 

이 글에서는 어떻게 제가 extension을 사용했는지 소개해보겠습니다. 아마 다음 글은 dart 3의 records로 리펙토링한 경험을 소개해드릴까 합니다.

> 제가 좋은 방식이라고 확신은 못합니다. 다른 좋은 방법으로 활용하시는 아이디어나 경험이 있으시면 댓글 남겨주시면 감사하겠습니다!

## Extension

### 사용 배경

img src="2023-10-06-flutter 주이어가 extension을 쓰는 법.assets/XL.jpeg" alt="쏙쏙 들어오는 함수형 코딩 - 예스24" style="zoom:25%;" 

 올 해 2월 중 읽었던 책중에, \<함수형 코딩\>이라는 책이 있었습니다. 함수형 프로그래밍은 side-effect를 최소화하여 쟐못된 방식으로 프로그램이 작동하는 것을 최대한 막고, 함수 실행 시점에서 변수의 상태를 비교적 정확하게 예측할 수 있는 프로그램 페러다임 중 하나 입니다.

사실 side-effect(부수효과)가 없는 프로그램은 없거나 거의 극소수일것입니다. 이 부수효과가 의미하는 것은 실제적으로 어떤 값을 변경하는것인데, 예를 들어서 메일 보낸다던가, 로켓을 실제로 쏘아올리는 프로세스의 끝과정 이 그 부수효과에 해당하는 것 입니다. flutter라고 하면 dialog를 만든다던가, 서버에 값을 보낸다던가 하는 게 되겠습니다.

따라서, 부수 효과를 잘 관리하는것이 함수형 프로그래밍의 핵심입니다. 그래서 함수형 프로그래밍은 코딩을 세 단계로 구분하는데, `액션`,`계산`,`데이터` 입니다. 

그래서 함수형 프로그래밍이 extension이랑 무슨 관계가 있는거냐... 저는 이 세 단계 중 '계산' 단계를 extension으로 리펙토링 있다고 생각했습니다.

### 사용 예시

1. Formatting

   - int나 double을 특정 string format으로 유저에게 보여주거나 서버에 전송 해야 할 때가 종종 있습니다.

   - 이럴 때 int에 extension을 걸어서 format을 해줄 수 있습니다.

   - ```dart
     extension IntX on int {
       String get asPriceFormat => NumberFormat("###,###,###원").format(this);
     }
     ```

   - datetime도 비슷하게 적용 가능합니다.

   - ``` dart
     extension DateTimeX on DateTime {
       String get yyyyMMdd => DateFormat("yyyy-MM-dd").format(this);
     }
     ```
   
2. function

   - 유용한 function 자체를 extension에 넣어서 사용할 수도 있습니다.

   - List에서 sorting하는 것을 구현 해 보았습니다.(record를 사용했습니다.)

     ```dart
     extension ListX on List {
       List<({A key, List<T> list})> groupAs<A, T>(
         A Function(T element) filter,
       ) {
         var result = <({A key, List<T> list})>[];
         for (final element in this) {
           var flag = true;
           for (var i = 0; i < result.length; i++) {
             if (result[i].key == filter(element)) {
               result[i].list.add(element);
               flag = false;
               continue;
             }
           }
           if (flag) {
             result.add((
               key: filter(element),
               list: [element],
             ));
           }
         }
         return result;
       }
     }
     ```

3. 코드 가독성 높이기

   - nullable 처리나 로직 자체를 readable하게 이름짓기.

   - 계산하기

     ```dart
     extension MenuX on Menu {
       bool get hasImage => menu_image != null && menu_thumbnail != null;
       bool get noImage => menu_image == null || menu_thumbnail == null;
       bool get is_inactive => !is_active;
       bool get hasRecommend => recommend_count > 0;
     }
     ```
     
     
