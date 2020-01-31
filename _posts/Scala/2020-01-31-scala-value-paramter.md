---
layout: single
classes:
  - wide
title: Scala - 클래스 생성자 파라미터를 멤버로 사용하기
comments: true
author_profile: true
tags:
  - scala


---

C++에선 클래스 생성자의 파라미터로 멤버를 초기화할 떄 아래와 같은 코드를 작성했을 것입니다.

~~~c++
class Foo
{
public:
  int myX, myY;
  Foo(int x, int y) :
    myX(x), myY(y)
  {
  }
}
~~~

멤버 변수를 미리 선언해두고, 생성자 파라미터의 값들을 가지고 초기화 리스트를 이용해 멤버 변수들을 초기화합니다.

한편, scala에서는 아래와 같이 멤버를 선언 및 초기화했을 것입니다.

~~~scala
class Foo(x: Int, y: Int) {
  val myX = x
  val myY = y
}
~~~

c++에서와 같이 멤버 변수를 미리 따로 선언할 필요 없이 클래스 body 내부에 선언과 동시에 초기화할 수 있습니다.

그러나 scala에서는 이 방법을 더욱 간단한 코드로 작성할 수 있습니다.
바로 scala에서는 생성자 파라미터를 바로 멤버로 사용할 수 있는 syntax를 제공하기 때문입니다.

아래 코드와 같이 작성해도 기존 코드와 동일한 동작을 합니다.

~~~scala
class Foo(val myX: Int, val myY: Int) {

}
~~~

단지 파라미터 앞에 `val`이란 키워드를 명시해주면 됩니다.
그러면 똑같이 `myX`, `myY`라는 멤버가 생기게 됩니다.