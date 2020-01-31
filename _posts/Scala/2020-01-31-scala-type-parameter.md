---
layout: single
classes:
  - wide
title: Scala - Type Parameter
comments: true
author_profile: true
tags:
  - scala



---

정수형(Integer)을 담고 있는 리스트를 구현한다고 했을 때, 값을 가지고 있는 노드와, 값이 없는 노드(보통 리스트의 끝을 나타내기 위함) 두 가지 종류의 노드가 있습니다.

그렇다면 구현할 때 노드는 공통적으로 `trait`으로 묶고, I해당 `trait`의 서브 클래스 두 개를 만들어 값을 가진 노드, 값이 없는 노드를 각각 표현할 수 있습니다.

값을 가진 노드 클래스를 `Cons`, 값이 없는 노드 클래스를 `Nil`이라고 하겠습니다.

~~~scala
trait IntList ...
class Cons(val head: Int, val tail: IntList) extends IntList ...
class Nil extends IntList ...
~~~

(`...`은 클래스의 body를 나타냅니다.)

그렇다면 1 -> 2 -> 3과 같은 리스트는 아래와 같이 시각화할 수 있습니다.

![](../../assets/images/list.png)



하지만 이렇게 구현한 클래스는 Int만 노드의 값으로 가질 수 있다는 단점이 있습니다.

다행히도, scala에서는 `type parameter`를 통해 Double, Boolean, 혹은 user defined class를 값으로 가지고 있는 리스트를 구현할 수 있습니다.

~~~scala
trait List[T] ...
class Cons[T](val head: T, val tail: List[T]) extends List[T] ...
class Nil[T] extends List[T] ...
~~~

클래스의 이름 뒤에 `[T]`를 붙여 여러 타입의 Cons, Nil을 만들 수 있습니다.

~~~scala
new Cons(1, new Nil)
~~~

이렇게 하면 Int 타입의 Cons 인스턴스가 생기고,

~~~scala
new Cons(false, new Nil)
~~~

이렇게 하면 Boolean 타입의 Cons 인스턴스가 생기게 됩니다.

이 `[T]`에 들어가는 타입은 호출자의 파라미터 타입을 컴파일 타임 때 컴파일러에 의해 추론(Inference)되어 확정되게 됩니다.

scala에서는 type parameter라는 것을 제공하므로, 다형성(Polymorphism)에서 generics가 가능한 language입니다.



**generics란?**

타입 파라미터화(type parameterization)에 의해 함수나 클래스의 인스턴스가 생성되는 것

저는 그냥 타입 파라미터에 의해 여러 타입의 클래스나 함수를 만들 수 있는 것이라고 이해하고 있습니다.