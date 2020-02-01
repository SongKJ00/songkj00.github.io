---
layout: single
classes:
  - wide
title: Scala - Covariance
comments: true
author_profile: true
tags:
  - scala




---

### Covariance

[이전 글 Type Bounds](https://songkj00.github.io/scala-type-bounds/)에 이어 작성하는 글입니다.

type bounds에 의하면 `NonEmpty <: IntSet` 이 참인 명제였습니다.

그렇다면 `List[NonEmpty] <: List[IntSet]`도 참이 될 수 있을까요?
즉, `NonEmpty`가 `IntSet`의 서브 타입이니 `List[NonEmpty]`도 `List[IntSet]`의 서브 타입일까요?

정답부터 얘기하자면 `List[NonEmpty]`는 `List[IntSet]`의 서브 타입입니다.

subtyping 관계는 type parameter 관계에 의해 결정됩니다.

이렇게 `NonEmpty`가 `IntSet`의 서브 타입일 때, `List[NonEmpty]`가 `List[IntSet]`의 서브 타입이 되는 것을 **covariance**라고 합니다.

그렇다면 covariance는 List 뿐만 아니라 모든 타입에도 적용될까요?



### Arrays

Java의 array를 살펴보겠습니다.

Java에서 `T` 타입의 요소를 가지는 array는 `T[]`라고 사용합니다.

Java에서 array는 list와 같이 covariant하기 때문에 `NonEmpty[] <: IntSet[]` 은 참입니다.



### Array Typing Problem

하지만, covariant한 array는 문제가 발생할 수 있습니다.

아래와 같은 자바 코드를 예로 들겠습니다.

~~~java
NonEmpty[] a = new NonEmpty[](new NonEmpty(1, Empty, Empty))
IntSet[] b = a
b[0] = Empty
NonEmpty s = a[0]
~~~

line-by-line으로 설명하면 아래와 같습니다.

1. NonEmpty 타입의 array `a`를 생성합니다.
2. IntSet 타입의 array `b`에 `a`를 할당합니다.
   이때, reference 할당이므로 또 다른 array를 만들어내진 않고 단순히 `a`를 참조하는 것 뿐입니다.
3. `b[0]`에 Empty를 할당합니다.
4. NonEmpty 타입 `s`에 `a[0]`을 할당합니다.

마지막 라인에 Empty 타입을 NonEmpty 타입을 할당합니다.

그러나 위 코드는 run-time error를 발생시킵니다.

첫번째, 두번째 라인은 문제 없이 잘 동작합니다.

하지만 세 번째 라인에서 ArrayStoreException이 발생합니다.

Java에서는 array를 만들 때 array의 타입에 대한 type tag를 붙입니다.
따라서 첫 번째 라인에서 NonEmpty array를 만들 때, 해당 array에 NonEmpty란 type tage를 붙입니다.

Array의 요소(element)에 무언가 할당할 때, 이 type tag를 확인합니다.

따라서, 세 번째 라인에서 Empty 타입을 할당하려 했지만 type tag를 확인해 보니 NonEmpty이므로 run-time error를 발생시키게 됩니다.

그렇다면 이런 run-time error를 방지하기 위해 특정 타입이 다른 타입의 서브 타입이 되어야 할 때는 언제고, 그렇지 않아야 할 때는 언제일까요?

### The Liskov Substitution Principle

Barbara Liskov가 얘기한 원칙은 언제 특정 타입이 다른 타입의 서브 타입이 될 수 있는지 얘기합니다.

> "만약 `A <: B`를 만족한다면, B 타입에서 가능한 것들 모두 A 타입에서도 가능해야 한다."

Liskov의 실제 정의는 이렇게 말합니다.

> `A <: B`일 때, `B` 타입으로 만든 object `x`에 대해 `q(x)`가 가능하다면, A 타입으로 만든 object `y`에 대해 `q(y)`가 가능해야 한다.

따라서 이 원칙은 '어느 동작(operation)을 수행할 수 있느냐'가 아니라 'object로 어떤 것을 증명(prove)할 수 있느냐'의 관점에서 얘기하고 있습니다.



### Array Typing Problem - Scala

Java에서 얘기한 Array Typing Problem 코드를 scala로 바꿔보겠습니다.

~~~scala
val a: Array[NonEmpty] = Array(new NonEmpty(1, Empty, Empty))
val b: Array[IntSet] = a
b(0) = Empty
val s: NonEmpty = a(0)
~~~

Java에서는 run-time error가 발생했고, scala에서도 아래와 같은 선택지 중 하나의 결과가 발생할 것입니다.

* compile 타임에 type error가 발생
* compile은 되지만, run-time exception 발생
* compile이 되고, run-time exception이 발생하지 않음

정답은 `compile 타임에 type error가 발생`입니다.

두 번째 라인 `val b: Array[IntSet] = a`에서 type error가 발생합니다. 이 코드에서 NonEmpty로 선언한 Array `a`를 IntSet으로 선언한 Array `b`에 할당하려 합니다.

두 번째 라인에서 type error가 발생한 이유는 scala에서 Java와 달리 Array를 covariance로 여기지 않기 때문입니다.

따라서 `Array[NonEmpty] <: Array[IntSet]`는 성립할 수 없습니다.