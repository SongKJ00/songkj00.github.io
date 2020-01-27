---
layout: single
classes:
  - wide
title: Scala - Operator
comments: true
author_profile: true
tags:
  - scala



---

[이 강의](https://www.coursera.org/lecture/progfun1/lecture-2-7-evaluation-and-operators-6WD2X)를 보고 정리한 글입니다.

이전에 유리수 클래스 `Rational`을 정의했습니다.

정수 x, y에 대해서 덧셈을 하기 위해선 `x + y`라는 코드를 사용하게 됩니다.
그러나 우리가 정의한 유리수 클래스로 만든 유리수끼리의 덧셈은 `r.add(s)`라는 코드로 바꿔서 사용해야 합니다.

scala에서는 둘의 차이를 없앨 수 있습니다.

### Infix Notation(중위 표현)

파라미터가 필요한 모든 메소드는 infix operator처럼 사용할 수 있습니다.

예를 들어, 
`r.add(s)`는 `r add s`로, 
`r.less(s)`는 `r less s`로,
`r.max(s)`는 `r max s`로 바꾸어서 사용할 수 있는 것이죠.



### Relaxed Identifiers

scala에서는 연산자(operator)가 식별자(identifier)처럼 사용될 수 있습니다.
scala에서 식별자로 사용 가능한 것은 아래와 같습니다.

* 영숫자(alphanumeric) : 영문자로 시작하여 뒤에 영문자나 숫자가 옴
* 기호(symbolic): 연산자 기호로 시작하여 뒤에 다른 연산자 기호가 옴
  * +, -, *, / 등과 같은 연산자 기호가 식별자로 사용 가능함
* 영숫자 식별자로 시작해서 underscore(_)로 끝난 뒤, 뒤에 기호 식별자가 오는 것도 허용
  * underscore 문자 '_'는 문자로 여겨짐

따라서 아래와 같은 식별자들이 가능합니다.

* x1
* *
* +?%&
* vector_++
* counter_=

### 실제 Rational 클래스 메소드 구현 바꿔보기

#### less 메소드

두 유리수의 크기를 비교하여 현재 유리수가 작으면 true를, 그 반대이면 false를 내뱉도록 구현된 `less` 메소드가 있었습니다.

~~~scala
def less (that: Rational) = 
  numer * that.denom < that.numer * denom
~~~

이제 이 메소드를 위에서 얘기했던 Infix notation과 relaxed identifiers에 의해 아래와 같이 바꿀 수 있습니다.

~~~scala
def < (that: Rational) = 
  numer * that.denom < that.numer * denom
~~~



#### max 메소드

`less` 메소드가 위와 같이 바뀌면 두 유리수 중 큰 유리수를 return하던 `max` 메소드의 구현도 바꿀 수 있습니다.

기존 `max` 메소드 구현

~~~scala
def max(that: Rational) = 
  if (this.less(that)) that else this
~~~

바뀐 max 메소드 구현

~~~scala
def max(that: Rational) = 
  if (this < that) that else this
~~~



#### add 메소드

두 유리수의 덧셈에 대한 메소드인 `add`도 바꿀 수 있습니다.
기존 `add` 메소드 구현

~~~scala
def add(that: Rational) = 
  new Rational(
    numer * that.denom + that.numer * denom,
    denom * that.denom)
~~~

바뀐 `add` 메소드 구현

~~~scala
def + (that: Rational) = 
  new Rational(
    numer * that.denom + that.numer * denom,
    denom * that.denom)
~~~

#### sub 메소드

`add` 메소드가 `+`로 바뀌었으니 `sub` 메소드도 바꿀 수 있습니다.
기존 `sub` 메소드 구현

~~~scala
def sub(that: Rational) = add(that.neg)
~~~

바뀐 `sub` 메소드 구현

~~~scala
def - (that: Rational) = this + that.neg
~~~

#### neg 메소드

현재 유리수의 반대 부호를 가지는 유리수를 return하던 neg 메소드도 바꿀 수 있습니다.
`that.neg` 이렇게 쓰는 것보다 `-that` 이렇게 쓰는 것이 수학적으로 좀 더 직관적으로 보일 수 있습니다.
이렇게 접두어 마이너스를 쓰기 위해서는 단순히 `def -`와 같이 구현할 수 없습니다.(infix operator가 아니기 때문)

따라서 앞에 단항을 나타내는 `unary_`를 붙여주어야 합니다.

기존 `neg` 메소드 구현

~~~scala
def neg: Rational = new Rational(-numer, denom)
~~~

바뀐 `neg` 메소드 구현

~~~scala
def unary_- : Rational = new Rational(-numer, denom)
~~~

`neg` 메소드가 위와 같이 바뀌면 `sub` 메소드도 아래와 같이 한번 더 바뀔 수 있습니다.

~~~scala
def - (that: Rational) = this + -that
~~~

메소드가 연산자 기호로 끝나는 경우, 콜론(:)과 공백이 있어야 합니다.
예로, 아래와 같이 `unary_-`와 `:`이 붙어있으면 안 됩니다.
만약 둘이 붙어있다면 콜론도 허용되는 기호이기 때문에 `unary_-:` 라는 이름의 메소드를 만들려고 할 것 입니다.

~~~scala
def unary_-: Rational = new Rational(-numer, denom)
~~~



따라서, 우리가 만든 유리수 클래스 Rational의 객체의 사칙연산도 이제 정수의 사칙연산처럼 다룰 수 있게 됩니다.

~~~scala
val x = new Rational(1, 2)
val y = new Rational(2, 3)
x + y
~~~

