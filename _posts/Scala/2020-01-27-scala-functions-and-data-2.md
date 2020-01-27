---
layout: single
classes:
  - wide
title: Scala - Functions and Data 2
comments: true
author_profile: true
tags:
  - scala


---

[이전 글](https://songkj00.github.io/scala-functions-and-data/)에 이어서 작성하는 글입니다.

### dry principle

~~~scala
val y = new Rational(5, 7)
println(y.add(y).toString)
~~~

이 결과는 무엇일까요? `5/7 + 5/7`이므로 당연히 10/7일 것입니다.
하지만 결과는 70/49입니다. 
약분되지 않은 결과가 나오게 된 것이죠.

위 예제에서는 유리수가 약분되지 않았습니다.
약분을 구현하기 위해서는 분자와 분모의 최대 공약수로 나누는 기능이 추가가 되어야 합니다.

약분 기능을 유리수의 모든 연산 메소드(덧셈, 뺄셈, 곱셈, 나눗셈)에 구현할 수 있지만, 이는 잊어버리기 쉽상이고 코드를 반복하지 말라는 **dry principle**에 위배될 수 있습니다.

그러므로, 약분을 구현하기 위한 더 좋은 방법은 객체가 생성될 때 해당 기능이 구현되는 것입니다.

아래와 같이 구현할 수 있습니다.

~~~scala
class Rational(x: Int, y: Int) {
  private def gcd(a: Int, b: Int): Int = if (b == 0) a else gcd(b, a % b)
  private val g = gcd(x, y)
  def numer = x / g
  def denom = y / g
  // ...
}
~~~

여기서 gcd는 두 정수의 최대 공약수를 구하는 함수입니다.

이 예제에서, `gcd`와 `g`는 private 멤버이므로 Rational 클래스 내부에서만 접근할 수 있습니다.
또한, 객체가 생성될 때 g를 바로 계산해 두고 numer와 denom에서 사용합니다.
그렇기에 우리가 numer와 denom을 사용할 때마다 g를 다시 계산할 필요가 없습니다.

위 코드가 추가되었다면, 이제 아래 코드의 결과는 70/49가 아닌 우리가 원하던 결과인 10/7이 출력될 것입니다.

~~~scala
val y = new Rational(5, 7)
println(y.add(y).toString)
~~~



한편, 예제에서, `numer`와 `denom`를 `val`로 바꾸어 한 번만 계산되도록 할 수 있습니다.
이 경우, numer와 denom이 자주 사용될 때 이점을 얻을 수 있습니다. 
왜냐하면 numer와 denom은 한 번만 계산되어 다시 또 계산되지 않기 때문이죠.

~~~scala
class Rational(x: Int, y: Int) {
  private def gcd(a: Int, b: Int): Int = if (b == 0) a else gcd(b, a % b)
  val numer = x / gcd(x, y)
  val denom = y / gcd(x, y)
  // ...
}
~~~



위 두 가지 중 어느 방법을 선택하든 client는 같은 동작을 예상합니다.
여기서, 어느 구현 방법(implementation)을 선택하든 client에게 영향을 미치지 않는 특징을 데이터 추상화(data abstraction)라고 합니다.
데이터 추상화는 소프트웨어 엔지니어링에 있어서 핵심 요소 중 하나입니다.

### this - 1

두 유리수의 크기를 비교하여 현재 유리수가 더 작으면 true를, 아니면 false를 return하는 메소드 `less`를 구현한다면, 아래와 같이 구현할 수 있습니다.

~~~scala
def less(that: Rational) = 
  numer * that.denom < that.numer * denom
~~~

그렇다면, 두 유리수 중 크기가 큰 유리수 자체를 return하는 메소드 `max`는 어떻게 구현해야 할까요?

~~~scala
def max(that: Rational) = 
  if (this.less(that)) that else this
~~~

여기서 this라는 키워드가 등장하는데, scala에서 `this`는 메소드가 실행되고 있는 객체를 나타냅니다.(현재 객체)

따라서, 메소드 `max`와 같이 현재 객체 전체를 return해야 할 때는 `this`가 필요합니다.

### this - 2

메소드 내부에서 `x`라는 이름을 쓰는 것은, `this.x`를 쓰는 것과 같습니다.(현재 객체의 멤버를 사용)
객체의 멤버는 `this`를 접두사로 하여 항상 참조할 수 있게 됩니다.

따라서 메소드 `less`는 아래 구현과 같습니다.

~~~scala
def less(that: Rational) = 
  this.numer * that.denom < that.numer * this.denom
~~~

위 메소드의 경우, 이렇게 this를 붙이면 좌우 대칭이 한 눈에 들어오게 됩니다.

### 예외 처리

분모를 0으로 가지는 Rational 객체 `strange`를 생성하고 해당 객체를 자기 자신에게 다시 더해보겠습니다.

~~~scala
val stange = new Rational(1, 0)
strange.add(strange)
~~~

그러면 아래와 같은 에러 메세지가 출력될 것입니다.

`java.lang.ArithmeticException: / by zero`

0을 분모로 두는 유리수는 당연히 없기 때문에 예외가 발생하게 됩니다.(정확히는 0으로 나누려 했기 때문에)

분모가 0으로 오는 것을 방지하기 위해선 어떻게 해야 할까요?

**require 사용**

~~~scala
class Rational(x: Int, y: Int) {
  require(y != 0, "denominator must be nonzero")
  // ...
}
~~~

위 코드를 추가한 뒤 실행하면 에러 메세지가 달라집니다.
`java.lang.IllegalArgumentException: requirement failed: denominator must be nonzero`

 scala에서 `require`이란 객체가 초기화될 때 수행되는 테스트이며, 해당 테스트가 실패하면 예외가 발생하게 됩니다.

`require`은 파라미터로 condition과 optional message string을 받게 되는데, condition이 false이면 optional message string과 함께 예외를 throw하게 됩니다.

**assert와 require의 차이**

require과 비슷해 보이는, 어쩌면 기존에 더 많이 썼던 assert라는 것이 있습니다.

assert 또한 파라미터로 condition과 optional message string을 받습니다.
아래 예제는 y의 제곱근이 0보다 같거나 커야 한다는 조건을 걸고 있고, 이 조건이 false이면 예외가 throw됩니다.

~~~scala
val x = sqrt(y)
assert(x >= 0)
~~~

assert와 require 모두 condition이 false이면 예외를 throw하지만, assert는 `AssertionError`라는 이름으로, require은 `IllegalArgumentException`이라는 이름의 예외를 throw합니다.

또한, require은 함수 호출이나 생성자의 사전 조건을 체크하기 위해 사용하고 assert는 함수 자체의 코드를 체크하기 위해 사용됩니다.

따라서, 사전 조건(잘못된 파라미터)이 실패하면 `IllegalArgumentException`이 발생하고 사전 조건의 실패가 아닌 다른 오류라면 `AssertionError`가 발생합니다.

**Constructor Overloading(생성자 오버로딩)**

자바와 마찬가지로, scala에서도 생성자 오버로딩을 구현할 수 있습니다.

현재 Rational 클래스의 객체를 생성하기 위해서는 두 개의 정수(분자, 분모)를 넣어주어야 합니다.

그렇다면 분자만 파라미터로 넘길 때는 분모가 1로 고정된다고 가정하면 매번 `new Rational(x, 1)`과 같이 호출해야 할까요?

이는 생성자 오버로딩을 통해서 간단하게 구현할 수 있습니다.

~~~scala
class Rational(x: Int, y: Int) {
  require(y != 0, "denominator must be nonzero")
  
  def this(x: Int) = this(x, 1)
  // ...
}
~~~

`def this(x: Int) = this(x, 1)` 코드를 추가하므로써 인자를 하나만 받는 생성자를 새롭게 정의하였습니다.
이 생성자는 파라미터로 받은 x를 가지고 기존 두 개의 정수를 파라미터로 받는 생성자를 호출하며 파라미터로 x와 1을 넘깁니다.

아래와 같은 코드로 해당 생성자를 이용하여 객체를 생성할 수 있습니다.

~~~scala
new Rational(2) // 2/1
~~~

