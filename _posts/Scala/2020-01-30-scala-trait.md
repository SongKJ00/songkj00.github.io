---
layout: single
classes:
  - wide
title: Scala - Trait
comments: true
author_profile: true
tags:
  - scala

---

Java뿐만 아니라 Scala에서도 단일 상속만 가능하기 때문에 클래스는 하나의 슈퍼클래스만 가질 수 있습니다.

그러나 클래스는 매우 많은 슈퍼 클래스(정확히는 super type)를 가져야 할 때가 있을 수도 있습니다.

이를 위해서 scala에는 `trait`라는 것이 있습니다.

`trait`은 Java의 `abstract class`와 거의 비슷하고, scala에서 `abstract class`라는 키워드 대신 `trait`라는 키워드로 단순하게 코드를 작성할 수 있습니다.

예로, 평면을 나타내는 타입을 구현한다고 할 때, 평면의 높이와 폭은 정의되지 않고, 평면의 넓이를 구하는 식만 (높이 * 폭)으로 기본 구현될 수 있습니다.

~~~scala
trait Planar {
  def height: Int
  def width: Int
  def surface = height * width
}
~~~

`surface` 메소드는 구현이 되어 있지만 나머지 두 개의 메소드는 구현되어 있지 않으므로 Planar는 추상적인 타입(abstract type)이라고 할 수 있습니다.

class와 object, 그리고 trait은 여러 개의 클래스를 상속할 수 없지만 여러 개의 trait을 상속(추가)할 수 있습니다.

예로 아래와 같이 `Square` 클래스가 `Shape`을 슈퍼 클래스로 두고 두 개의 `trait`(Planar와 Movable)을 추가하는 것이 가능합니다.
`trait`을 추가할 때는 `with` 키워드를 사용해야 합니다.

~~~scala
class Square extends Shape with Planar with Movable ...
~~~

`trait`은 Java의 `interface`와 되게 유사해보이지만, `trait`은 필드(변수)와 비추상 메소드(구현이 된  메소드)를 포함할 수 있다는 차이점이 있습니다.

하지만 `trait`는 결국 클래스가 아니기 때문에 마찬가지로 생성자를 가질 수 없어 인스턴스화를 할 수 없습니다.

다시 Planar 예제를 들고 오겠습니다.

~~~scala
trait Planar {
  def height: Int
  def width: Int
  def surface = height * width
}
~~~

이 Planar trait의 `surface` 메소드는 기본 구현을 제공합니다.
따라서 만약 Planar를 상속하는(추가하는) 서브 클래스에서 `surface` 메소드를 새롭게 재정의할 수 있지만, 재정의하지 않는다면 Planar의 `surface` 메소드가 사용되게 됩니다.