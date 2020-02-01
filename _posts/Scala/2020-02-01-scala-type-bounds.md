---
layout: single
classes:
  - wide
title: Scala - Type Bounds
comments: true
author_profile: true
tags:
  - scala



---

Type Bounds를 한 문장으로 정의한다면, `함수의 파라미터를 상속과 관련된 특정 범위로 제한할 수 있는 것`이라고 저는 정의하고 있습니다.

그러나 이 문장만 본다면 도저히 이해가 안 가므로 아래 예시를 들며 설명하겠습니다.

### Type Bounds

`assertAllPos` 메소드를 정의해본다고 하겠습니다.
이 메소드는 3가지의 요구사항을 가집니다.

* `IntSet`을 파라미터로 받습니다.
  `IntSet`은 user defined class입니다.
* 파라미터의 모든 요소들이 양수(positive)라면 파라미터로 들어온 값(`IntSet` 타입)을 return합니다.
* 만약 모든 요소들이 양수가 아니라면 exception을 throw합니다.

`assertAllPos`는 아래와 같이 정의할 수 있습니다.

~~~scala
def assertAllPos(s: IntSet): IntSet
~~~

이와 같은 정의는 대부분의 케이스에 대해 만족하겠지만, 더 정확하게 구현하려면 어떻게 해야 할까요?

IntSet을 상속하는 두 개의 클래스가 있습니다.
하나는 값을 가지고 있지 않은 클래스인 `Empty` 클래스, 그리고 값을 가지고 있는 클래스인 `NonEmpty` 클래스입니다.

그러면 `assertAllPos`는 두 개의 방정식(equation)을 가질 수 있습니다.

* assertAllPos(Empty) = Empty
* assertAllPos(NonEmpty(...)) = NonEmpty(...)
                                                  혹은
                                                  throw exception

Empty 타입이 파라미터로 들어오면 Empty 타입을 반환하고, NonEmpty 타입이 파라미터로 들어오면 NonEmpty 타입을 반환합니다.

여기서 하나 알아야 할 점은 실제 `assertAllPos`의 정의 즉, 실제 코드에는 `IntSet` 타입을 받는다고 나와있지, 서브 클래스인 `Empty`나 `NonEmpty`를 받는다고 나와있지 않습니다.

그렇다면 `Empty`를 받으면 `Empty`를 return하고, `NonEmpty`를 받으면 `NonEmpty`를 return한다고 어떻게 명시적으로 표현(express)할 수 있을까요?

assertAllPos(Empty) = Empty,
assertAllPos(NonEmpty) = NonEmpty

이렇게 두 개의 함수를 만들어야 할까요?

scala에서는 아래와 같은 하나의 함수로 이를 해결할 수 있습니다.

~~~scala
def assertAllPos[S <: IntSet](r: S): S = ...
~~~

여기서 `S`는 `IntSet`을  따르는(상속한) 타입으로만 인스턴스화할 수 있습니다.
즉, IntSet이나 Empty, NonEmpty 타입만 파라미터로 올 수 있다는 것이죠.

이 범위를 제한하는 게 바로 **"<: IntSet"** 이 부분입니다.
이 **"<: IntSet"**을 타입 파라미터 `S`의 **upper bound**라고 합니다.

이 bound의 의미를 다시 정리하면 아래와 같습니다.

* "S <: T" : S는 T의 서브 타입입니다.
* "S >: T" : S는 T의 슈퍼 타입입니다. &rarr; T는 S의 서브 타입입니다.

### Lower Bounds

`[S >: NonEmpty]`

위와 같은 코드는 타입 파라미터 `S`는 `NonEmpty`의 슈퍼 타입들만 가능하다는 의미입니다.

따라서 `S`는 `NonEmpty`, `IntSet`, `AnyRef`, `Any`만 가능합니다.

### Mixed Bounds

또한, lower bound와 upper bound를 같이 사용하는 것도 가능합니다.

`[S >: NonEmpty <: IntSet]`

위 코드는 타입 파라미터 `S`는 `NonEmpty`와 `IntSet` 사이에 있는 타입만 가능하다는 의미입니다.
따라서, `NonEmpty`, `IntSet`만 가능합니다.



### Conclusion

그렇다면 이 type bounds를 써서 가지는 이점은 무엇일까요?

여러 가지 파라미터 타입을 가지는 함수를 만들고자 하면 이전에 설명한 type parameter만으로도 충분히 가능했습니다.

그러나 여기서 type bounds 개념을 추가한 이유는 type parameter의 범위를 제한할 수 있기 때문이라고 생각합니다.

A
&uarr;
B
&uarr;
C

위와 같은 상속 구조에서 우리는 어떤 함수의 type paramter로 A와 B만 와야 하거나, C만 와야 하거나, 혹은 A와 B와 C만 오고 다른 user defined class로 만들어진 object는 피해야 할 경우가 있을 것입니다.

이럴 때 type bounds를 사용해서 클래스들의 상속 관계에서 특정 클래스들만 파라미터로 올 수 있게 할 수 있습니다.

그리고, 코드를 봐도 딱 "아 어느 타입부터 어느 타입까지만 파라미터로 올 수 있는 함수구나"라고 느낄 수 있어 좀 더 편하지 않을까 싶습니다.